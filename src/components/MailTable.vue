<template>
  <table class="mail-table">
    <tbody>
      <tr v-for="email in unarchivedEmails" 
          :key="email.id"
          :class="[email.read ? 'read': '', 'clickable']"
          @click="openEmail(email)">
        <td>
          <input type="checkbox" />
        </td>
        <td>{{email.from}}</td>
        <td>
          <p><strong>{{email.subject}}</strong> - {{email.body}}</p>
        </td>
        <td class="date">{{format(new Date(email.sentAt), 'MMM do yyyy')}}</td>
        <td><button @click="archiveEmail(email)">Archive</button></td>
      </tr>
    </tbody>
  </table>
  <ModalView v-if="openedEmail" @closeModal="closeModal">
    <MailView :email="openedEmail"></MailView>
  </ModalView>
</template>

<script>
import { format } from 'date-fns';
import axios from 'axios';
import MailView from "@/components/MailView.vue";
import ModalView from "@/components/ModalView.vue";

export default {
  name: 'MailTable',
  components: {
    MailView,
    ModalView
  },
  async setup(){
    let { data: emails } = await axios.get('http://localhost:3000/emails');
    return {
      format,
      emails,
      openedEmail: null
    }
  },
  methods: {
    openEmail(email) {
      email.read = true;
      this.updateEmail(email);
      this.openedEmail = email;
    },
    archiveEmail(email) {
      email.archived = true;
      this.updateEmail(email);
    },
    closeModal() {
      this.openedEmail = null;
      this.$forceUpdate();
    },
    updateEmail(email) {
      axios.put(`http://localhost:3000/emails/${email.id}`, email);
      this.$forceUpdate();
    }
  },
  computed: {
    sortedEmails(){
      return this.emails.sort((e1, e2) => {
        return e1.sentAt < e2.sentAt ? 1 : -1
      });
    },
    unarchivedEmails(){
      return this.sortedEmails.filter(e => !e.archived);
    }
  }
}
</script>
<template>
  <button
    @click="selectScreen('inbox')"
    :disabled="selectedScreen === 'inbox'"
  >
    Inbox
  </button>
  <button
    @click="selectScreen('archive')"
    :disabled="selectedScreen === 'archive'"
  >
    Archive
  </button>
  <BulkActionBar :emails="filteredEmails"></BulkActionBar>
  <table class="mail-table">
    <tbody>
      <tr v-for="email in filteredEmails" 
          :key="email.id"
          :class="[email.read ? 'read': '', 'clickable']">
        <td>
          <input
            type="checkbox"
            @click="emailSelection.toggle(email)"
            :checked="emailSelection.emails.has(email)"
          />
        </td>
        <td @click="openEmail(email)">{{email.from}}</td>
        <td @click="openEmail(email)">
          <p><strong>{{email.subject}}</strong> - {{email.body}}</p>
        </td>
        <td class="date" @click="openEmail(email)">
          {{format(new Date(email.sentAt), 'MMM do yyyy')}}
        </td>
        <td>
          <button
            v-if="!email.archived"
            @click="toggleArchiveEmail(email)"
          >
            Archive
          </button>
          <button
            v-if="email.archived"
            @click="toggleArchiveEmail(email)"
          >
            Unarchive
          </button>
        </td>
      </tr>
    </tbody>
  </table>
  <ModalView v-if="openedEmail" @closeModal="closeModal">
    <MailView :email="openedEmail" @changeEmail="changeEmail"></MailView>
  </ModalView>
</template>

<script>
import { reactive, ref } from "vue";
import { format } from 'date-fns';
import axios from 'axios';
import MailView from "@/components/MailView.vue";
import ModalView from "@/components/ModalView.vue";
import BulkActionBar from "@/components/BulkActionBar.vue";
import useEmailSelection from "@/composables/use-email-selection";

export default {
  name: 'MailTable',
  components: {
    MailView,
    ModalView,
    BulkActionBar
  },
  async setup(){
    let { data: emails } = await axios.get('http://localhost:3000/emails');

    return {
      format,
      emails: ref(emails),
      openedEmail: ref(null),
      emailSelection: useEmailSelection(),
      selectedScreen: ref('inbox')
    }
  },
  methods: {
    selectScreen(newScreen) {
      this.selectedScreen = newScreen;
      this.emailSelection.clear();
    },
    openEmail(email) {
      this.openedEmail = email;
      if(email) {
        email.read = true;
        this.updateEmail(email);
      }
    },
    toggleArchiveEmail(email) {
      email.archived = !email.archived;
      this.updateEmail(email);
    },
    closeModal() {
      this.openedEmail = null;
    },
    changeEmail({toggleRead, toggleArchive, save, closeModal, changeIndex}) {
      let email = this.openedEmail;
      if(toggleRead) { email.read = !email.read }
      if(toggleArchive) { email.archived = !email.archived }
      if(save) { this.updateEmail(email) }
      if(closeModal) { this.closeModal() }
      if(changeIndex) {
        let emails = this.unarchivedEmails;
        let currentIndex = emails.indexOf(this.openedEmail);
        let newIndex = currentIndex + changeIndex;
        let newEmail = emails[newIndex];
        this.openEmail(newEmail);
      }
    },
    updateEmail(email) {
      axios.put(`http://localhost:3000/emails/${email.id}`, email);
    }
  },
  computed: {
    sortedEmails(){
      return this.emails.sort((e1, e2) => {
        return e1.sentAt < e2.sentAt ? 1 : -1
      });
    },
    filteredEmails(){
      return this.selectedScreen === "inbox"
        ? this.sortedEmails.filter(e => !e.archived)
        : this.sortedEmails.filter(e => e.archived);
    },
  }
}
</script>
<template>
  <div>
    <NewMeetingForm @added="addNewMeeting($event)"></NewMeetingForm>
      <div :class="'alert alert-' + (this.isError ? 'error' : 'success')" v-if="message">{{ message }}</div>
    <span v-if="meetings.length == 0">Brak zaplanowanych spotkań.</span>
    <h3 v-else>Zaplanowane zajęcia ({{ meetings.length }})</h3>
    <MeetingsList :meetings="meetings"
                  :username="username"
                  @attend="addMeetingParticipant($event)"
                  @unattend="removeMeetingParticipant($event)"
                  @delete="deleteMeeting($event)"></MeetingsList>
  </div>
</template>

<script>
import NewMeetingForm from "./NewMeetingForm";
import MeetingsList from "./MeetingsList";
import axios from "axios";

export default {
  components: {NewMeetingForm, MeetingsList},
  props: {username: String},
  data() {
      return {
          meetings: [],
       id: '',
          message: '',
          isError: false,
    }
    },
  created(){
        this.getMeetings()
  },

    methods: {
        getMeetings(){
            axios.get('/api/meetings').then(response => {
                for (let i =0; i <response.data.length;i++) {
                   this.getParticipants(response.data[i])
                }
                this.meetings = response.data
            })
                .catch(error => this.failure(`Błąd przy pobieraniu spotkań. Szczegóły błędu: ${error.response.data}`));
        },

        getParticipants(meeting) {
            meeting.participants=[]
            axios.get('/api/meetings', {
                params: {
                    title: meeting.title,
                    description: meeting.description
                }
            }).then(response => {
                this.id = response.data[0].id
                const url = "/api/meetings/" + this.id + "/participants"
                axios.get(url)
                    .then(response => {
                        for (let i =0; i <response.data.length;i++) {
                            meeting.participants.push(response.data[i].login)
                        }
                    })
                    .catch(error => this.failure(`Błąd przy pobieraniu uczestników. Szczegóły błędu: ${error.response.data}`));
            })
                .catch(error => this.failure(`Błąd przy pobieraniu spotkania. Szczegóły błędu: ${error.response.data}`));
        },

    addNewMeeting(meeting) {
        axios.post('/api/meetings', meeting)
            .then(() => {
                this.getMeetings()
            })
            .catch(error => this.failure(`Błąd przy dodawaniu spotkania. Szczegóły błędu: ${error.response.data}`));
    },

    addMeetingParticipant(meeting) {
    axios.get('/api/meetings', {params: {title: meeting.title, description: meeting.description}}).then(response => {
            this.id = response.data[0].id
            const url = "/api/meetings/" + this.id + "/participants"
            axios.post(url, {login: this.username})
                .then(() => {
                    this.getParticipants(meeting)
                })
                .catch(error => this.failure(`Błąd przy dodawaniu uczestnika. Szczegóły błędu: ${error.response.data}`));
    }).catch(error => this.failure(`Błąd przy wyszukaniu spotkania. Szczegóły błędu: ${error.response.data}`));
    },

    removeMeetingParticipant(meeting) {
        axios.get('/api/meetings', {params: {title: meeting.title, description: meeting.description}}).then(response => {
            this.id = response.data[0].id
            axios.delete('/api/meetings/' + this.id + '/participants/' + this.username).then(() => {
                this.getMeetings()
            })
                .catch(error => this.failure(`Błąd przy usuwaniu uczestnika. Szczegóły błędu: ${error.response.data}`));
        })
            .catch(error => this.failure(`Błąd przy wyszukaniu spotkania. Szczegóły błędu: ${error.response.data}`));

    },
    deleteMeeting(meeting) {
        axios.get('/api/meetings', {params: {title: meeting.title, description: meeting.description}}).then(response => {
            this.id = response.data[0].id
            axios.delete('/api/meetings/' + this.id).then(() => {
                this.getMeetings()
            })
                .catch(error => this.failure(`Błąd przy usuwaniu spotkania. Szczegóły błędu: ${error.response.data}`));
        })
            .catch(error => this.failure(`Błąd przy wyszukaniu spotkania. Szczegóły błędu: ${error.response.data}`));

    },
        failure(message) {
            this.message = message;
            this.isError = true;
        },
        clearMessage() {
            this.message = undefined;
        },
  }
}
</script>

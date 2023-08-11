<template>
  <div class="event">
    <div class="block">
      <el-form-item label="Event">
        <el-input
          :readonly="readonly"
          v-model="event"
          type="text"
          placeholder="Event Title"
        />
      </el-form-item>
    </div>
    <div class="block">
      <el-form-item label="Name">
        <el-input
          :readonly="readonly"
          v-model="name"
          type="text"
          placeholder="Name"
        />
      </el-form-item>
    </div>
    <div class="block">
      <el-form-item label="Email">
        <el-input
          :readonly="readonly"
          v-model="email"
          type="text"
          placeholder="email"
        />
      </el-form-item>
    </div>
    <div class="block">
      <el-form-item label="Description">
        <el-input
          :readonly="readonly"
          v-model="description"
          type="textarea"
          placeholder="Description"
        />
      </el-form-item>
    </div>
    <div class="block">
      <el-form-item label="Recipients">
        <el-input
          :readonly="readonly"
          v-model="recipients"
          type="textarea"
          placeholder="name@email.com, name2@email.com..."
        />
      </el-form-item>
    </div>

    <div class="block">
      <el-form-item label="Date">
        <el-date-picker
          :readonly="readonly"
          v-model="date"
          type="date"
          placeholder="Select date and time"
        />
      </el-form-item>
    </div>

    <div v-if="date" class="block">
      <el-form-item label="Time">
        <el-time-select
          v-if="!readonly"
          v-model="time"
          start="08:30"
          step="00:15"
          end="18:30"
          placeholder="Select time"
        />
        <el-input
          v-if="readonly"
          :readonly="readonly"
          v-model="time"
        />
      </el-form-item>
    </div>

    <div class="block">
      <el-button @click="setAppointment" type="primary">Download</el-button>
      <a
        class="el-button el-button--primary"
        v-if="link"
        :href="link"
        target="_blank"
        >send mail</a
      >
    </div>
  </div>
</template>

<script>
import { ref } from 'vue';
import { createEvent } from 'ics';

const date = ref('');
const time = ref('');
const endTime = ref('');
const event = ref('');
const description = ref('');
const email = ref('');
const name = ref('');
const recipients = ref('');
const location = ref('');
const link = ref('');
const readonly = ref(false);

export default {
  name: 'EventForm',
  props: {
    msg: String,
  },
  data() {
    return {
      date,
      time,
      endTime,
      event,
      description,
      email,
      name,
      recipients,
      location,
      link,
      readonly,
    };
  },
  methods: {
    async setAppointment() {
      const time = this.time.split(':');
      const event = {
        start: [
          this.date.getFullYear(),
          this.date.getMonth() + 1,
          this.date.getDate(),
          +time[0],
          +time[1],
        ],
        duration: { hours: 6, minutes: 30 },
        title: this.event,
        description: this.description,
        location: this.location,
        // url: 'http://www.bolderboulder.com/',
        // geo: { lat: 40.0095, lon: 105.2669 },
        // categories: ['10k races', 'Memorial Day Weekend', 'Boulder CO'],
        // status: 'CONFIRMED',
        // busyStatus: 'BUSY',
        organizer: { name: this.name, email: this.email },
        attendees: this.recipients.split(',').map((r) => ({
          name: r.split('@')[0],
          email: r.trim(),
          rsvp: true,
        })),
        // attendees: [
        //   { name: 'Adam Gibbons', email: 'adam@example.com', rsvp: true, partstat: 'ACCEPTED', role: 'REQ-PARTICIPANT' },
        //   { name: 'Brittany Seaton', email: 'brittany@example2.org', dir: 'https://linkedin.com/in/brittanyseaton', role: 'OPT-PARTICIPANT' }
        // ]
      };

      const filename = 'ExampleEvent.ics';
      const file = await new Promise((resolve, reject) => {
        createEvent(event, (error, value) => {
          if (error) {
            reject(error);
          }

          resolve(new File([value], filename, { type: 'plain/text' }));
        });
      });
      const url = URL.createObjectURL(file);

      // trying to assign the file URL to a window could cause cross-site
      // issues so this is a workaround using HTML5
      const anchor = document.createElement('a');
      anchor.href = url;
      anchor.download = filename;

      document.body.appendChild(anchor);
      anchor.click();
      document.body.removeChild(anchor);

      URL.revokeObjectURL(url);

      const eventS = {
        event: this.event,
        name: this.name,
        email: this.email,
        description: this.description,
        location: this.location,
        recipients: this.recipients,
        date: this.date,
        start: this.time,
        end: this.endTime,
      };
      this.link = encodeURI(
        `mailto:${this.recipients}?subject=${this.event}&body=${`${
          this.event
        } \n ${this.description} \n ${this.location} \n ${this.date} \n ${
          this.time
        } - ${this.endTime} \n Follow Link to set calendar: https://${
          window.location.hostname
        }?event=${encodeURI(JSON.stringify(eventS))}`}`
      );
    },
  },
  mounted() {
    // debuging
    // const test = {
    //   event: 'test',
    //   name: 'jo',
    //   email: 'email@test.com',
    //   description: 'this is test',
    //   location: 'BFE',
    //   recipients: 'email2@test.com, email3@test.com',
    //   date: new Date(),
    //   start: '9:30',
    //   end: '10:30',
    // };

    const urlSearchParams = new URLSearchParams(window.location.search);
    let params = Object.fromEntries(urlSearchParams.entries());

    // Debugging
    // params = params.event ? JSON.parse(params.event) : test;
    if (params.event) {
      params = JSON.parse(params.event);
      this.event = params.event;
      this.name = params.name;
      this.email = params.email;
      this.description = params.description;
      this.location = params.location;
      this.recipients = params.recipients;
      this.date = new Date(params.date);
      this.time = params.start;
      this.endTime = params.end;
      this.readonly = true;
      this.setAppointment();
    }
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped></style>

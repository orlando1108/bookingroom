<template>
  <div class="home" style="margin-bottom:50px">
    <b-container>
      <img alt="Vue logo" src="../assets/logo2.png" width="200px">
      <!-- <HelloWorld msg="Welcome to Your Vue.js App"/>-->
      <planning
        class="vuecal--green-theme vuecal--full-height-delete"
        :on-event-create="createEvent"
        @event-delete="deleteEvent"
        :hideViewSelector="true"
        :selected-date="selectedDate"
        :events="events"
        :disable-views="['years', 'year', 'month','day']"
        editable-events
        hide-weekends
        default-view="week"
        locale="fr"
        :time-from="startTime * 60"
        :time-to="endTime * 60"
        :time-step="30"
      ></planning>
      <b-modal
        id="ajout-horaire"
        :title="getDateToday()"
        v-model="modalShow"
        hide-header-close
        centered
        no-close-on-backdrop
        header-bg-variant="dark"
        header-text-variant="white"
      >
        <b-form-group
          :label-cols="4"
          label-align="right"
          description
          label="Heure de debut"
          label-for="time-options"
        >
          <b-form-select
            id="time-options"
            v-model="form.selectedTime"
            :options="timeOptions"
            :change="isCorrectTimeSlot"
          />
        </b-form-group>
        <b-form-group
          :label-cols="4"
          label-align="right"
          description
          label="Durée"
          label-for="duration-options"
        >
          <b-form-select
            id="duration-options"
            v-model="form.selectedDuration"
            :options="durationOptions"
            :change="isCorrectTimeSlot"
          />
        </b-form-group>

        <b-form-group label-cols="4" label-align="right" label="Entreprise" label-for="company">
          <b-form-select
            id="company"
            v-model="form.selectedCompany"
            :options="companyOptions"
            :change="isCorrectTimeSlot"
          />
        </b-form-group>

        <div slot="modal-footer" class="w-100">
          <b-container fluid>
            <b-row>
              <b-col>
                <b-button
                  class="float-right"
                  size="sm"
                  variant="danger"
                  block
                  @click="cancelEventCreation()"
                >Annuler</b-button>
              </b-col>
              <b-col>
                <transition name="bounce">
                  <b-button
                    v-if="isCorrectTimeSlot"
                    class="float-right"
                    size="sm"
                    variant="success"
                    block
                    @click="addEvent()"
                  >Ajouter</b-button>
                </transition>
              </b-col>
            </b-row>
          </b-container>
        </div>
      </b-modal>
    </b-container>
  </div>
</template>

<script>
// @ is an alias to /src
import HelloWorld from "@/components/HelloWorld.vue";
import planning from "vue-cal";
//import moment from "moment";
import "vue-cal/dist/vuecal.css";

const moment = require("moment-timezone");
moment.locale("fr");

export default {
  name: "home",
  components: {
    HelloWorld,
    planning
  },
  data() {
    return {
      events: [],
      selectedDate: "",
      selectedEvent: {},
      modalShow: false,
      durationOptions: [
        { value: null, text: "choisir" },
        { value: 1, text: "1 Heure" },
        { value: 2, text: "2 Heures" },
        { value: 4, text: "4 Heures" }
      ],
      companyOptions: [
        { value: null, text: "choisir" },
        { value: "AJP", text: "AJP" },
        { value: "Koumoul", text: "Koumoul" },
        { value: "EdgeMind", text: "EdgeMind" }
      ],
      timeOptions: [],
      form: {
        selectedTime: null,
        selectedDuration: null,
        selectedCompany: null,
        selectedTime: null
      },
      startTime: 8,
      endTime: 19
    };
  },
  mounted() {
    this.createTimeOptions();
    this.addLunchEvent();
  },
  computed: {
    isCorrectTimeSlot() {
      return this.form.selectedTime && this.form.selectedDuration;
    },
    modalTitle() {
      return "Réservation pour le" + this.selectedDate;
    }
  },
  methods: {
    createEvent(event, deleteEventFunction) {
      console.log(JSON.stringify(event));
      this.selectedDate = event.startDate;
      this.selectedTime = event.startTime;
      /*this.timeOptions[0].value = event.startTime;
      this.timeOptions[0].text = event.startTime;*/
      this.selectedEvent = event;
      this.modalShow = !this.modalShow;
      this.deleteEventFunction = deleteEventFunction;

      return event;
    },
    deleteEvent(event) {},
    addEvent() {
      let selectedDate = moment(this.selectedDate)
          .tz("Europe/Paris")
          .valueOf(),
        dateBegin = moment(this.selectedDate)
          .tz("Europe/Paris")
          .set("hour", this.form.selectedTime)
          .valueOf(),
        dateEnd = moment(dateBegin)
          .tz("Europe/Paris")
          .add(this.form.selectedDuration, "hours")
          .valueOf();

      this.selectedEvent = {
        start: moment(dateBegin).format("YYYY-MM-DD HH:mm"),
        end: moment(dateEnd).format("YYYY-MM-DD HH:mm"),
        class: "planning_" + this.form.selectedCompany,
        title: this.form.selectedCompany,
        date: selectedDate,
        ...this.form
      };
      this.events.push(this.selectedEvent);
      this.closeEventModal();
    },
    cancelEventCreation() {
      this.closeEventModal();
      this.deleteEventFunction();
      this.resetForm();
    },
    closeEventModal() {
      this.modalShow = false;
      this.selectedEvent = {};
    },
    createTimeOptions() {
      let options = [{ value: null, text: "choisir" }];
      let actualHour = this.isToday()
        ? moment()
            .tz("Europe/Paris")
            .format("H")
        : this.startTime;
      for (let i = actualHour; i <= this.endTime; i++) {
        options.push({
          value: i,
          text: i + " heure"
        });
      }
      this.timeOptions = options;
      // return options;
    },
    resetForm() {
      /* Reset our form values */
      this.form.timeSelected = null;
      this.form.dureeSelected = null;
      /* Trick to reset/clear native browser form validation state */
      //this.show = false;
      /*this.$nextTick(() => {
        this.show = true;
      });*/
    },
    getDateToday() {
      return moment(this.selectedDate)
        .tz("Europe/Paris")
        .format("LL");
    },
    isToday() {
      let selected = moment(this.selectedDate)
        .tz("Europe/Paris")
        .startOf("day")
        .valueOf();
      let today = moment()
        .tz("Europe/Paris")
        .startOf("day")
        .valueOf();
      return selected === today;
    },
    addLunchEvent() {
      const today = moment().isoWeekday();
      let days=[];
      let currentDay = 1;
      while (currentDay <= 7) {
        const dayOfWeek = moment().isoWeekday(currentDay).format("YYYY-MM-DD");
        console.log(dayOfWeek);
        const lunchEvent={
          start: dayOfWeek+' 12:00',
          end: dayOfWeek+' 13:30',
          title: 'DÎNER',
          class: 'lunch',
          background: true
        }
        this.events.push(lunchEvent);
        console.log(this.events)
        currentDay++;
      }
      //console.log('days' + days);
    }
  }
};
</script>
<style lang="scss">
.vuecal__event.planning_AJP {
  background-color: #DC6BAD;
  color: rgb(255, 255, 255);
  border-width: 1px;
  border-style: solid;
  border-color:  #DC6BAD;
}
.vuecal__event.planning_Koumoul {
  background-color: #8C7AA9;
  color: rgb(255, 255, 255);
  border-width: 1px;
  border-style: solid;
  border-color: #8C7AA9;
}
.vuecal__event.planning_EdgeMind {
  background-color: #D76F47;
  color: rgb(255, 255, 255);
  border-width: 1px;
  border-style: solid;
  border-color:#D76F47;
}
.vuecal__no-event {
  display: none;
}
.vuecal__event.lunch {
  background: repeating-linear-gradient(45deg, transparent, transparent 10px, #f2f2f2 10px, #f2f2f2 20px);/* IE 10+ */
  color: #999;
  display: flex;
  justify-content: center;
  align-items: center;
}
.vuecal__event.lunch .vuecal__event-time {display: none;align-items: center;}
//.vuecal__time-cell .hours.line:before {border-color: #42b983;}
/*
AJP: #DC6BAD
KOUMOUL: 8C7AA9
edgemind D36135 / D76F47
*/
</style>

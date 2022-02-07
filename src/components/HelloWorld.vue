<template>
  <b-container fluid>
    <!-- User Interface controls -->
    <b-row v-if="false">
      <b-col lg="6" class="my-1">
        <b-form-group
          label="Sort"
          label-for="sort-by-select"
          label-cols-sm="3"
          label-align-sm="right"
          label-size="sm"
          class="mb-0"
          v-slot="{ ariaDescribedby }"
        >
          <b-input-group size="sm">
            <b-form-select
              id="sort-by-select"
              v-model="sortBy"
              :options="sortOptions"
              :aria-describedby="ariaDescribedby"
              class="w-75"
            >
              <template #first>
                <option value="">-- none --</option>
              </template>
            </b-form-select>

            <b-form-select
              v-model="sortDesc"
              :disabled="!sortBy"
              :aria-describedby="ariaDescribedby"
              size="sm"
              class="w-25"
            >
              <option :value="false">Asc</option>
              <option :value="true">Desc</option>
            </b-form-select>
          </b-input-group>
        </b-form-group>
      </b-col>

      <b-col lg="6" class="my-1">
        <b-form-group
          label="Initial sort"
          label-for="initial-sort-select"
          label-cols-sm="3"
          label-align-sm="right"
          label-size="sm"
          class="mb-0"
        >
          <b-form-select
            id="initial-sort-select"
            v-model="sortDirection"
            :options="['asc', 'desc', 'last']"
            size="sm"
          ></b-form-select>
        </b-form-group>
      </b-col>

      <b-col lg="6" class="my-1">
        <b-form-group
          label="Filter"
          label-for="filter-input"
          label-cols-sm="3"
          label-align-sm="right"
          label-size="sm"
          class="mb-0"
        >
          <b-input-group size="sm">
            <b-form-input
              id="filter-input"
              v-model="filter"
              type="search"
              placeholder="Type to Search"
            ></b-form-input>

            <b-input-group-append>
              <b-button :disabled="!filter" @click="filter = ''"
                >Clear</b-button
              >
            </b-input-group-append>
          </b-input-group>
        </b-form-group>
      </b-col>

      <b-col lg="6" class="my-1">
        <b-form-group
          v-model="sortDirection"
          label="Filter On"
          description="Leave all unchecked to filter on all data"
          label-cols-sm="3"
          label-align-sm="right"
          label-size="sm"
          class="mb-0"
          v-slot="{ ariaDescribedby }"
        >
          <b-form-checkbox-group
            v-model="filterOn"
            :aria-describedby="ariaDescribedby"
            class="mt-1"
          >
            <b-form-checkbox value="name">Name</b-form-checkbox>
            <b-form-checkbox value="age">Age</b-form-checkbox>
            <b-form-checkbox value="isActive">Active</b-form-checkbox>
          </b-form-checkbox-group>
        </b-form-group>
      </b-col>

      <b-col sm="5" md="6" class="my-1">
        <b-form-group
          label="Per page"
          label-for="per-page-select"
          label-cols-sm="6"
          label-cols-md="4"
          label-cols-lg="3"
          label-align-sm="right"
          label-size="sm"
          class="mb-0"
        >
          <b-form-select
            id="per-page-select"
            v-model="perPage"
            :options="pageOptions"
            size="sm"
          ></b-form-select>
        </b-form-group>
      </b-col>

      <b-col sm="7" md="6" class="my-1">
        <b-pagination
          v-model="currentPage"
          :total-rows="totalRows"
          :per-page="perPage"
          align="fill"
          size="sm"
          class="my-0"
        ></b-pagination>
      </b-col>
    </b-row>

    <!-- Main table element -->
    <b-table
      :items="items"
      :fields="fields"
      :current-page="currentPage"
      :per-page="perPage"
      :filter="filter"
      :filter-included-fields="filterOn"
      :sort-by.sync="sortBy"
      :sort-desc.sync="sortDesc"
      :sort-direction="sortDirection"
      stacked="md"
      show-empty
      small
      @filtered="onFiltered"
    >
      <template #cell(actions)="row">
        <b-button size="sm" @click="row.toggleDetails">
          {{ row.detailsShowing ? "Hide" : "Show" }} Details
        </b-button>
      </template>

      <template #row-details="row">
        <b-card>
          <b-table :items="parse(row)" :fields="subFields">
            <template #cell(actions)="row">
              <b-button size="sm" @click="row.toggleDetails">
                {{ row.detailsShowing ? "Hide" : "View" }} More
              </b-button>
            </template>
            <template #row-details="row">
              <b-card>
                <b-table :items="parseDetails(row)" :fields="batchFields">
                  <template #cell(time)="row">
                    {{ row.item.start_time }} - {{ row.item.end_time }}
                  </template>
                </b-table>
              </b-card>
            </template>
          </b-table>
          
           <b-table :items="parseStudent(row)" :fields="studentFields">
            <template #cell(actions)="row">
              <b-button size="sm" @click="row.toggleDetails">
                {{ row.detailsShowing ? "Hide" : "View" }} More
              </b-button>
            </template>
            <template #row-details="row">
              <b-card>
                <b-table :items="parseDetails(row)" :fields="batchFields">
                  <template #cell(time)="row">
                    {{ row.item.start_time }} - {{ row.item.end_time }}
                  </template>
                </b-table>
              </b-card>
            </template>
          </b-table>
        </b-card>
      </template>
      
    </b-table>

    <!-- Info modal -->
    <b-modal
      :id="infoModal.id"
      :title="infoModal.title"
      ok-only
      @hide="resetInfoModal"
    >
      <pre>{{ infoModal.content }}</pre>
    </b-modal>
  </b-container>
</template>

<script>
import Vue from 'vue'
import { BootstrapVue, IconsPlugin } from 'bootstrap-vue'

// Import Bootstrap an BootstrapVue CSS files (order is important)
import 'bootstrap/dist/css/bootstrap.css'
import 'bootstrap-vue/dist/bootstrap-vue.css'

// Make BootstrapVue available throughout your project
Vue.use(BootstrapVue)
// Optionally install the BootstrapVue icon components plugin
Vue.use(IconsPlugin)


export default {

  data() {
    return {
      items: [
        {
          name: "Priya Gupta",
          phone: "998877445",
          email: "priya.gupta@gmail.com",
          average_rate_per_class: 350,
          total_classes_completed: 15,
          payment_earned: 3500,
          tatal_batches: 5,
          total_students: 12,
          classes_information: [
            {
              type: "WEBINAR",
              class_name: "German A1.1-Adults",
              total_classes_completed: 10,
              per_class_rate: 200,
              payment_earned: 2000,
              call_information: [
                {
                  topic_name: "Gretting",
                  date: "12 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:30 PM",
                  status: "completed",
                },
                {
                  topic_name: "German Grammer",
                  date: "13 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:30 PM",
                  status: "completed",
                },
                {
                  topic_name: "German Culture",
                  date: "14 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:30 PM",
                  status: "rescheduled",
                },
                {
                  topic_name: "Basics",
                  date: "15 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:30 PM",
                  status: "update_pending",
                },
                {
                  topic_name: "Famous",
                  date: "16 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:30 PM",
                  status: "scheduled",
                },
              ],
            },
            {
              type: "SESSION",
              student_name: "Ananya Gupta",
              total_classes_completed: 10,
              per_class_rate: 200,
              payment_earned: 2000,
              call_information: [
                {
                  topic_name: "Gretting",
                  date: "12 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:00 PM",
                  status: "completed",
                },
                {
                  topic_name: "German Grammer",
                  date: "13 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:00 PM",
                  status: "completed",
                },
                {
                  topic_name: "German Culture",
                  date: "14 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:00 PM",
                  status: "rescheduled",
                },
                {
                  topic_name: "Basics",
                  date: "15 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:00 PM",
                  status: "update_pending",
                },
                {
                  topic_name: "Famous",
                  date: "16 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:00 PM",
                  status: "scheduled",
                },
              ],
            },
          ],
        },
        {
          name: "Mahesh",
          phone: "998877445",
          email: "mahesh@gmail.com",
          average_rate_per_class: 350,
          total_classes_completed: 15,
          payment_earned: 3500,
          tatal_batches: 5,
          total_students: 12,
          classes_information: [
            {
              type: "WEBINAR",
              class_name: "German A1.1-Adults",
              total_classes_completed: 10,
              per_class_rate: 200,
              payment_earned: 2000,
              call_information: [
                {
                  topic_name: "Gretting",
                  date: "12 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:30 PM",
                  status: "completed",
                },
                {
                  topic_name: "German Grammer",
                  date: "13 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:30 PM",
                  status: "completed",
                },
                {
                  topic_name: "German Culture",
                  date: "14 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:30 PM",
                  status: "rescheduled",
                },
                {
                  topic_name: "Basics",
                  date: "15 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:30 PM",
                  status: "update_pending",
                },
                {
                  topic_name: "Famous",
                  date: "16 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:30 PM",
                  status: "scheduled",
                },
              ],
            },
            {
              type: "SESSION",
              student_name: "Ananya Gupta",
              total_classes_completed: 10,
              per_class_rate: 200,
              payment_earned: 2000,
              call_information: [
                {
                  topic_name: "Gretting",
                  date: "12 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:00 PM",
                  status: "completed",
                },
                {
                  topic_name: "German Grammer",
                  date: "13 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:00 PM",
                  status: "completed",
                },
                {
                  topic_name: "German Culture",
                  date: "14 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:00 PM",
                  status: "rescheduled",
                },
                {
                  topic_name: "Basics",
                  date: "15 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:00 PM",
                  status: "update_pending",
                },
                {
                  topic_name: "Famous",
                  date: "16 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:00 PM",
                  status: "scheduled",
                },
              ],
            },
          ],
        },
        {
          name: "manjeeth",
          phone: "998877445",
          email: "manjeeth@gmail.com",
          average_rate_per_class: 50,
          total_classes_completed: 10,
          payment_earned: 500,
          tatal_batches: 5,
          total_students: 12,
          classes_information: [
            {
              type: "WEBINAR",
              class_name: "German A1.1-Adults",
              total_classes_completed: 10,
              per_class_rate: 200,
              payment_earned: 2000,
              call_information: [
                {
                  topic_name: "Gretting",
                  date: "12 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:30 PM",
                  status: "completed",
                },
                {
                  topic_name: "German Grammer",
                  date: "13 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:30 PM",
                  status: "completed",
                },
                {
                  topic_name: "German Culture",
                  date: "14 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:30 PM",
                  status: "rescheduled",
                },
                {
                  topic_name: "Basics",
                  date: "15 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:30 PM",
                  status: "update_pending",
                },
                {
                  topic_name: "Famous",
                  date: "16 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:30 PM",
                  status: "scheduled",
                },
              ],
            },
            {
              type: "SESSION",
              student_name: "Ananya Gupta",
              total_classes_completed: 10,
              per_class_rate: 200,
              payment_earned: 2000,
              call_information: [
                {
                  topic_name: "Gretting",
                  date: "12 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:00 PM",
                  status: "completed",
                },
                {
                  topic_name: "German Grammer",
                  date: "13 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:00 PM",
                  status: "completed",
                },
                {
                  topic_name: "German Culture",
                  date: "14 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:00 PM",
                  status: "rescheduled",
                },
                {
                  topic_name: "Basics",
                  date: "15 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:00 PM",
                  status: "update_pending",
                },
                {
                  topic_name: "Famous",
                  date: "16 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:00 PM",
                  status: "scheduled",
                },
              ],
            },
          ],
        },
        {
          name: "Priya Gupta",
          phone: "998877445",
          email: "priya.gupta@gmail.com",
          average_rate_per_class: 350,
          total_classes_completed: 15,
          payment_earned: 3500,
          tatal_batches: 5,
          total_students: 12,
          classes_information: [
            {
              type: "WEBINAR",
              class_name: "German A1.1-Adults",
              total_classes_completed: 10,
              per_class_rate: 200,
              payment_earned: 2000,
              call_information: [
                {
                  topic_name: "Gretting",
                  date: "12 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:30 PM",
                  status: "completed",
                },
                {
                  topic_name: "German Grammer",
                  date: "13 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:30 PM",
                  status: "completed",
                },
                {
                  topic_name: "German Culture",
                  date: "14 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:30 PM",
                  status: "rescheduled",
                },
                {
                  topic_name: "Basics",
                  date: "15 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:30 PM",
                  status: "update_pending",
                },
                {
                  topic_name: "Famous",
                  date: "16 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:30 PM",
                  status: "scheduled",
                },
              ],
            },
            {
              type: "SESSION",
              student_name: "Ananya Gupta",
              total_classes_completed: 10,
              per_class_rate: 200,
              payment_earned: 2000,
              call_information: [
                {
                  topic_name: "Gretting",
                  date: "12 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:00 PM",
                  status: "completed",
                },
                {
                  topic_name: "German Grammer",
                  date: "13 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:00 PM",
                  status: "completed",
                },
                {
                  topic_name: "German Culture",
                  date: "14 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:00 PM",
                  status: "rescheduled",
                },
                {
                  topic_name: "Basics",
                  date: "15 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:00 PM",
                  status: "update_pending",
                },
                {
                  topic_name: "Famous",
                  date: "16 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:00 PM",
                  status: "scheduled",
                },
              ],
            },
          ],
        },
        {
          name: "Mahesh",
          phone: "998877445",
          email: "mahesh@gmail.com",
          average_rate_per_class: 350,
          total_classes_completed: 15,
          payment_earned: 3500,
          tatal_batches: 5,
          total_students: 12,
          classes_information: [
            {
              type: "WEBINAR",
              class_name: "German A1.1-Adults",
              total_classes_completed: 10,
              per_class_rate: 200,
              payment_earned: 2000,
              call_information: [
                {
                  topic_name: "Gretting",
                  date: "12 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:30 PM",
                  status: "completed",
                },
                {
                  topic_name: "German Grammer",
                  date: "13 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:30 PM",
                  status: "completed",
                },
                {
                  topic_name: "German Culture",
                  date: "14 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:30 PM",
                  status: "rescheduled",
                },
                {
                  topic_name: "Basics",
                  date: "15 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:30 PM",
                  status: "update_pending",
                },
                {
                  topic_name: "Famous",
                  date: "16 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:30 PM",
                  status: "scheduled",
                },
              ],
            },
            {
              type: "SESSION",
              student_name: "Ananya Gupta",
              total_classes_completed: 10,
              per_class_rate: 200,
              payment_earned: 2000,
              call_information: [
                {
                  topic_name: "Gretting",
                  date: "12 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:00 PM",
                  status: "completed",
                },
                {
                  topic_name: "German Grammer",
                  date: "13 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:00 PM",
                  status: "completed",
                },
                {
                  topic_name: "German Culture",
                  date: "14 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:00 PM",
                  status: "rescheduled",
                },
                {
                  topic_name: "Basics",
                  date: "15 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:00 PM",
                  status: "update_pending",
                },
                {
                  topic_name: "Famous",
                  date: "16 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:00 PM",
                  status: "scheduled",
                },
              ],
            },
          ],
        },
        {
          name: "manjeeth",
          phone: "998877445",
          email: "manjeeth@gmail.com",
          average_rate_per_class: 50,
          total_classes_completed: 10,
          payment_earned: 500,
          tatal_batches: 5,
          total_students: 12,
          classes_information: [
            {
              type: "WEBINAR",
              class_name: "German A1.1-Adults",
              total_classes_completed: 10,
              per_class_rate: 200,
              payment_earned: 2000,
              call_information: [
                {
                  topic_name: "Gretting",
                  date: "12 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:30 PM",
                  status: "completed",
                },
                {
                  topic_name: "German Grammer",
                  date: "13 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:30 PM",
                  status: "completed",
                },
                {
                  topic_name: "German Culture",
                  date: "14 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:30 PM",
                  status: "rescheduled",
                },
                {
                  topic_name: "Basics",
                  date: "15 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:30 PM",
                  status: "update_pending",
                },
                {
                  topic_name: "Famous",
                  date: "16 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:30 PM",
                  status: "scheduled",
                },
              ],
            },
            {
              type: "SESSION",
              student_name: "Ananya Gupta",
              total_classes_completed: 10,
              per_class_rate: 200,
              payment_earned: 2000,
              call_information: [
                {
                  topic_name: "Gretting",
                  date: "12 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:00 PM",
                  status: "completed",
                },
                {
                  topic_name: "German Grammer",
                  date: "13 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:00 PM",
                  status: "completed",
                },
                {
                  topic_name: "German Culture",
                  date: "14 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:00 PM",
                  status: "rescheduled",
                },
                {
                  topic_name: "Basics",
                  date: "15 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:00 PM",
                  status: "update_pending",
                },
                {
                  topic_name: "Famous",
                  date: "16 Jan 2022",
                  start_time: "01:30 PM",
                  end_time: "02:00 PM",
                  status: "scheduled",
                },
              ],
            },
          ],
        },
      ],
      fields: [
        {
          label: "Name",
          key: "name",
          sortable: true,
          class: "text-center",
        },
        {
          label: "Mobile Number",
          key: "phone",
        },
        {
          label: "Email",
          key: "email",
        },
        {
          label: "Per Class Rate",
          key: "average_rate_per_class",
        },
        {
          label: "Total Classes",
          key: "total_classes_completed",
        },
        {
          label: "Payment Earned",
          key: "payment_earned",
        },
        {
          label: "Total Batches",
          key: "tatal_batches",
        },
        {
          label: "Total Students",
          key: "total_students",
        },
        { key: "actions", label: "Actions" },
      ],
      subFields: [
        {
          label: "Batch",
          key: "class_name",
        },
        {
          label: "Total Classes",
          key: "total_classes_completed",
        },
        {
          label: "Per Class Rate",
          key: "per_class_rate",
        },

        {
          label: "Payment Earned",
          key: "payment_earned",
        },

        { key: "actions", label: "View More" },
      ],
      batchFields: [
        {
          label: "Topic Name",
          key: "topic_name",
        },
        {
          label: "Status",
          key: "status",
        },
        {
          label: "Time",
          key: "time",
        },

        {
          label: "Date",
          key: "date",
        },
        {
          label: "Payment Earned",
          key: "payment_earned"
        }
      ],
      
      studentFields: [
        {
          label: "Student",
          key: "student_name",
        },
        {
          label: "Target Language",
          key: "status",
        },
        {
          label: "Total Classes Completed",
          key: "total_classes_completed",
        },

        {
          label: "Per Class Rate",
          key: "per_class_rate",
        },
        {
          label: "Payment Earned",
          key: "payment_earned",
        },
                { key: "actions", label: "Actions" },
      ],

      totalRows: 1,
      currentPage: 1,
      perPage: 5,
      pageOptions: [5, 10, 15, { value: 100, text: "Show a lot" }],
      sortBy: "",
      sortDesc: false,
      sortDirection: "asc",
      filter: null,
      filterOn: [],
      infoModal: {
        id: "info-modal",
        title: "",
        content: "",
      },
    };
  },
  computed: {
    sortOptions() {
      // Create an options list from our fields
      return this.fields
        .filter((f) => f.sortable)
        .map((f) => {
          return { text: f.label, value: f.key };
        });
    },
  },
  mounted() {
    // Set the initial number of items
    this.totalRows = this.items.length;
  },
  methods: {
    info(item, index, button) {
      this.infoModal.title = `Row index: ${index}`;
      this.infoModal.content = JSON.stringify(item, null, 2);
      this.$root.$emit("bv::show::modal", this.infoModal.id, button);
    },
    parse(str) {
      const obj = JSON.parse(JSON.stringify(str));

      console.log(obj.item);

      console.log(obj.item.classes_information);

      return obj.item.classes_information.filter( function(obj){
        if(obj.type == 'WEBINAR'){
            return obj;
        }
    });
    },
    parseStudent(str) {
      const obj = JSON.parse(JSON.stringify(str));

      console.log(obj.item);

      console.log(obj.item.classes_information);

      return obj.item.classes_information.filter( function(obj){
        if(obj.type == 'SESSION'){
            return obj;
        }
    });
    },
    
    parseDetails(str) {
      const obj = JSON.parse(JSON.stringify(str));
      console.log(obj);
      return obj.item.call_information;
    },
    resetInfoModal() {
      this.infoModal.title = "";
      this.infoModal.content = "";
    },
    onFiltered(filteredItems) {
      // Trigger pagination to update the number of buttons/pages due to filtering
      this.totalRows = filteredItems.length;
      this.currentPage = 1;
    },
  },
};
</script>
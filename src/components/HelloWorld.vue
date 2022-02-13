<template>
  <b-container fluid>
    <!-- User Interface controls -->
    <b-row>
      <b-col>
        <h3>Trainer Payment</h3>
      </b-col>
      <b-col cols="7">

      </b-col>
      <b-col cols="2">
        <b-button variant="outline-primary" @click="openFilterTaggle"> 
          Filter
        </b-button>
      </b-col>
    </b-row>
    <b-row class="pb-4" v-if="openFilter">
      <b-col lg="4" class="my-1">
        <b-form-group
          label="Search by Teacher"
          label-for="filter-input"
          label-cols-sm="3"
          label-align-sm="right"
          label-size="sm"
          class="mb-0"
        >
          <b-input-group size="sm">
            <b-form-input
              id="filter-input"
              v-model="filterTeacher"
              type="search"
              placeholder="Type to Search"
            ></b-form-input>

            <b-input-group-append>
              <b-button :disabled="!filterTeacher" @click="filterTeacher = ''"
                >Clear</b-button
              >
            </b-input-group-append>
          </b-input-group>
        </b-form-group>
      </b-col>
      <b-col lg="4" class="my-1">
        <b-form-group
          label="Search by user"
          label-for="filter-input"
          label-cols-sm="3"
          label-align-sm="right"
          label-size="sm"
          class="mb-0"
        >
          <b-input-group size="sm">
            <b-form-input
              id="filter-input"
              v-model="filterUser"
              type="search"
              placeholder="Type to Search"
            ></b-form-input>

            <b-input-group-append>
              <b-button :disabled="!filterUser" @click="filterUser = ''"
                >Clear</b-button
              >
            </b-input-group-append>
          </b-input-group>
        </b-form-group>
      </b-col>
      <b-col lg="4" class="my-1">
        <b-form-group
          label="Cell Status"
          label-for="filter-input"
          label-cols-sm="3"
          label-align-sm="right"
          label-size="sm"
          class="mb-0"
        >
          <b-input-group size="sm">
            <b-form-input
              id="filter-input"
              v-model="filterStatus"
              type="search"
              placeholder="Type to Search"
            ></b-form-input>

            <b-input-group-append>
              <b-button :disabled="!filterStatus" @click="filterStatus = ''"
                >Clear</b-button
              >
            </b-input-group-append>
          </b-input-group>
        </b-form-group>
      </b-col>

      <b-col lg="4" class="my-1">
        <b-form-group
          label="Target Language"
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
          </b-input-group>
        </b-form-group>
      </b-col>

      <b-col lg="4" class="my-1">
        <p>Ava per class rate</p>

        <div class="range-slider">
          <span class="rangeValues">{{ rangeValues }}</span>
          <input
            min="0"
            max="1000"
            step="200"
            v-on:change="getVals"
            v-model="slide1"
            type="range"
          />
          <input
            min="0"
            max="1000"
            step="200"
            type="range"
            v-on:change="getVals"
            v-model="slide2"
          />
        </div>
      </b-col>

      <b-col lg="4" class="my-1">
        <div>
          <input type="date" v-model="startDate" v-on:change="dateFilter" />
          <input type="date" v-model="endDate" v-on:change="dateFilter" />
        </div>
      </b-col>
    </b-row>

    <!-- Main table element -->
    <b-table
      :items="items"
      :fields="fields"
      :current-page="currentPage"
      :per-page="perPage"
      :filter="filterTeacher"
      :filter-included-fields="filterOn"
      :sort-by.sync="sortBy"
      :filter-function="filterTable"
      :sort-desc.sync="sortDesc"
      :sort-direction="sortDirection"
      stacked="md"
      borderless
      show-empty
      small
      @filtered="onFiltered"
    >
      <template #cell(name)="row">
        <b-button
          size="sm"
          variant="outline-primary"
          @click="row.toggleDetails"
        >
          {{ row.item.name }}{{ row.detailsShowing ? " ▲" : " ▼" }}
        </b-button>
      </template>

      <template #row-details="row">
        <b-card>
          <b-row>
            <b-col cols="4">
              <b-card border-variant="success" class="m-4" text-variant="dark">
                <b-card-text>
                 Your total earning for the period are Rs.{{row.item.payment_earned }} from {{row.item.total_classes_completed}} completed classes.
                </b-card-text>
              </b-card>
            </b-col>
            
            <b-col cols="4">
              <b-card bg-variant="light" class="m-4" text-variant="dark">
                <b-card-text>
                  Update status for {{row.item.update_pending_calsses}} classes and you might add upto Rs.{{row.item.update_pending_amount}}
                  more in your earning.
                </b-card-text>
              </b-card>
            </b-col>
            <b-col cols="4">
              <b-card border-variant="danger" class="m-4" text-variant="dark">
                <b-card-text>
                 You have lost Rs.{{row.item.lost_amount}} amount in this period beacuse you rescheduled {{row.item.rescheduled_calsses }} of your classes.
                </b-card-text>
              </b-card>
            </b-col>
          </b-row>

          <b-table
            :items="parse(row)"
            :fields="subFields"
            :filter="filterUser"
            :filter-included-fields="filterOn"
            @filtered="onFiltered"
          >
            <template #cell(actions)="row">
              <b-button
                size="sm"
                variant="outline-primary"
                @click="row.toggleDetails"
              >
                View {{ row.detailsShowing ? " Less ▲" : " More ▼" }}
              </b-button>
            </template>
            <template #row-details="row">
              <b-card>
                <b-table
                  :items="parseDetails(row)"
                  :fields="batchFields"
                  :filter="filterStatus"
                  :filter-included-fields="filterOn"
                  @filtered="onFiltered"
                >
                  <template #cell(time)="row">
                    {{ row.item.start_time }} - {{ row.item.end_time }}
                  </template>
                </b-table>
              </b-card>
            </template>
          </b-table>

          <b-table
            :items="parseStudent(row)"
            :fields="studentFields"
            :filter="filterStatus"
            :filter-included-fields="filterOn"
            @filtered="onFiltered"
          >
            <template #cell(actions)="row">
              <b-button
                size="sm"
                variant="outline-primary"
                @click="row.toggleDetails"
              >
                View {{ row.detailsShowing ? " Less ▲" : " More ▼" }}
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
    <b-row>
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
  </b-container>
</template>

<script>
import Vue from "vue";
import { BootstrapVue, IconsPlugin } from "bootstrap-vue";

// Import Bootstrap an BootstrapVue CSS files (order is important)
import "bootstrap/dist/css/bootstrap.css";
import "bootstrap-vue/dist/bootstrap-vue.css";

// Make BootstrapVue available throughout your project
Vue.use(BootstrapVue);
// Optionally install the BootstrapVue icon components plugin
Vue.use(IconsPlugin);

export default {
  data() {
    return {
      
      tempItems: [],
      items: [
        {
          name: "Priya Gupta",
          phone: "998877445",
          email: "priya.gupta@gmail.com",
          average_rate_per_class: 350,
          total_classes_completed: 15,
          payment_earned: 3500,
          update_pending_calsses:6,
          update_pending_amount:1200,
          lost_amount:850,
          rescheduled_calsses:3,
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
          
          update_pending_calsses:6,
          update_pending_amount:1200,
          lost_amount:850,
          rescheduled_calsses:3,
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
          
          update_pending_calsses:6,
          update_pending_amount:1200,
          lost_amount:850,
          rescheduled_calsses:3,
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
          
          update_pending_calsses:6,
          update_pending_amount:1200,
          lost_amount:850,
          rescheduled_calsses:3,
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
          
          update_pending_calsses:6,
          update_pending_amount:1200,
          lost_amount:850,
          rescheduled_calsses:3,
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
          
          update_pending_calsses:6,
          update_pending_amount:1200,
          lost_amount:850,
          rescheduled_calsses:3,
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
          sortable: true,
        },
        {
          label: "Total Classes",
          key: "total_classes_completed",
          sortable: true,
        },
        {
          label: "Payment Earned",
          key: "payment_earned",
          sortable: true,
        },
        {
          label: "Total Batches",
          key: "tatal_batches",
          sortable: true,
        },
        {
          label: "Total Students",
          key: "total_students",
          sortable: true,
        },
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

        { key: "actions", label: "" },
      ],
      batchFields: [
        {
          label: "Topic Name",
          key: "topic_name",
        },
        {
          label: "Date",
          key: "date",
        },
        {
          label: "Time",
          key: "time",
        },

        {
          label: "Status",
          key: "status",
          tdClass: this.cellVerified,
        },
        {
          label: "Payment Earned",
          key: "payment_earned",
        },
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
        { key: "actions", label: "" },
      ],
      startDate: null,
      endDate: null,
      totalRows: 1,
      currentPage: 1,
      perPage: 5,
      pageOptions: [5, 10, 15, { value: 100, text: "Show a lot" }],
      sortBy: "",
      sortDesc: false,
      sortDirection: "asc",
      filter: null,
      filterTeacher: null,
      filterUser: null,
      filterStatus: null,
      filterOn: [],
      rangeValues: null,
      slide2: 1000,
      slide1: 0,
      openFilter: false,
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
  created() {
    this.tempItems = this.items;
  },
  mounted() {
    // Set the initial number of items
    this.totalRows = this.items.length;
  },
  methods: {
    filterTable(row) {
      console.log(row);
      // if (row.average_rate_per_class >= this.slide1 && row.average_rate_per_class <= this.slide2 ) {
      //   return true;
      // } else {
      //   return false;
      // }
    },
    openFilterTaggle() {
      this.openFilter = !this.openFilter;
    },
    getVals() {
      // Get slider values

      // Neither slider will clip the other, so make sure we determine which is larger
      if (this.slide1 > this.slide2) {
        let tmp = this.slide2;
        this.slide2 = this.slide1;
        this.slide1 = tmp;
      }
      this.items = this.tempItems.filter(
        (x) =>
          x.average_rate_per_class >= this.slide1 &&
          x.average_rate_per_class <= this.slide2
      );

      this.rangeValues = "$" + this.slide1 + " - $" + this.slide2;
    },
    dateFilter() {
      // Get slider values

      console.log(this.items);
      this.items = this.tempItems.filter((x) =>
        x.classes_information[0].cell_information.filter(
          (x = x.date >= this.startDate && x.date <= this.endDate)
        )
      );
    },

    function() {
      // Initialize Sliders
      let sliderSections = document.getElementsByClassName("range-slider");
      for (let x = 0; x < sliderSections.length; x++) {
        let sliders = sliderSections[x].getElementsByTagName("input");
        for (let y = 0; y < sliders.length; y++) {
          if (sliders[y].type === "range") {
            sliders[y].oninput = this.getVals;
            // Manually trigger event first time to display values
            sliders[y].oninput();
          }
        }
      }
    },
    cellVerified(row) {
      return row == "completed"
        ? "text-success"
        : row == "rescheduled"
        ? "text-warning"
        : "text-primary";
    },

    parse(str) {
      const obj = JSON.parse(JSON.stringify(str));

      console.log(obj.item);

      console.log(obj.item.classes_information);

      return obj.item.classes_information.filter(function (obj) {
        if (obj.type == "WEBINAR") {
          return obj;
        }
      });
    },
    parseStudent(str) {
      const obj = JSON.parse(JSON.stringify(str));

      console.log(obj.item);

      console.log(obj.item.classes_information);

      return obj.item.classes_information.filter(function (obj) {
        if (obj.type == "SESSION") {
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

<style>
.range-slider {
  width: 300px;
  text-align: center;
  position: relative;
}
.rangeValues {
  display: block;
}
input[type="range"] {
  -webkit-appearance: none;
  border: 1px solid white;
  width: 300px;
  position: absolute;
  left: 0;
}

input[type="range"]::-webkit-slider-runnable-track {
  width: 300px;
  height: 5px;
  background: #ddd;
  border: none;
  border-radius: 3px;
}

input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none;
  border: none;
  height: 16px;
  width: 16px;
  border-radius: 50%;
  background: #21c1ff;
  margin-top: -4px;
  cursor: pointer;
  position: relative;
  z-index: 1;
}

input[type="range"]:focus {
  outline: none;
}

input[type="range"]:focus::-webkit-slider-runnable-track {
  background: #ccc;
}

input[type="range"]::-moz-range-track {
  width: 300px;
  height: 5px;
  background: #ddd;
  border: none;
  border-radius: 3px;
}

input[type="range"]::-moz-range-thumb {
  border: none;
  height: 16px;
  width: 16px;
  border-radius: 50%;
  background: #21c1ff;
}

/*hide the outline behind the border*/

input[type="range"]:-moz-focusring {
  outline: 1px solid white;
  outline-offset: -1px;
}

input[type="range"]::-ms-track {
  width: 300px;
  height: 5px;
  /*remove bg colour from the track, we'll use ms-fill-lower and ms-fill-upper instead */
  background: transparent;
  /*leave room for the larger thumb to overflow with a transparent border */
  border-color: transparent;
  border-width: 6px 0;
  /*remove default tick marks*/
  color: transparent;
  z-index: -4;
}

input[type="range"]::-ms-fill-lower {
  background: #777;
  border-radius: 10px;
}

input[type="range"]::-ms-fill-upper {
  background: #ddd;
  border-radius: 10px;
}

input[type="range"]::-ms-thumb {
  border: none;
  height: 16px;
  width: 16px;
  border-radius: 50%;
  background: #21c1ff;
}

input[type="range"]:focus::-ms-fill-lower {
  background: #888;
}

input[type="range"]:focus::-ms-fill-upper {
  background: #ccc;
}
</style>
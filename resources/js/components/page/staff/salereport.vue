<template>
  <v-layout class="justify-end mt-4 mr-5 mb-3">
    <v-flex xs9 sm11>
      <v-card>
        <v-container fluid grid-list-md>
          <v-card-title>
            <v-layout row wrap justify-space-between>
              <h2 class="txt-title mt-0">
                <v-icon large color="pink">description</v-icon>
                &nbsp;รายงานยอดขาย ({{pageShow}})
              </h2>
              <v-flex xs12 sm5 md4>
                <v-layout row wrap>
                  <v-select
                    v-model="is_day"
                    :items="typeRep"
                    item-text="title"
                    label="เลือกดูประเภทรายงาน"
                    required
                  ></v-select>
                  <v-dialog
                    ref="dialog"
                    v-model="modal"
                    :return-value.sync="date"
                    v-if="is_day"
                    persistent
                    lazy
                    full-width
                    width="290px"
                  >
                    <template v-slot:activator="{ on }">
                      <v-text-field v-model="date" prepend-icon="event" readonly v-on="on"></v-text-field>
                    </template>
                    <v-date-picker
                      v-model="date"
                      color="pink accent-3"
                      locale="th"
                      type="date"
                      scrollable
                    >
                      <v-spacer></v-spacer>
                      <v-btn flat color="primary" @click="modal = false">ยกเลิก</v-btn>
                      <v-btn flat color="primary" @click="$refs.dialog.save(date)">ตกลง</v-btn>
                    </v-date-picker>
                  </v-dialog>
                  <v-dialog
                    ref="dialog"
                    v-model="modal"
                    :return-value.sync="month"
                    v-if="!is_day"
                    persistent
                    lazy
                    full-width
                    width="290px"
                  >
                    <template v-slot:activator="{ on }">
                      <v-text-field v-model="month" prepend-icon="event" readonly v-on="on"></v-text-field>
                    </template>
                    <v-date-picker
                      v-model="month"
                      color="pink accent-3"
                      locale="th"
                      type="month"
                      scrollable
                    >
                      <v-spacer></v-spacer>
                      <v-btn flat color="primary" @click="modal = false">ยกเลิก</v-btn>
                      <v-btn flat color="primary" @click="$refs.dialog.save(month)">ตกลง</v-btn>
                    </v-date-picker>
                  </v-dialog>
                  <v-btn color="primary" @click="reportPdf()">พิมพ์รายงาน</v-btn>
                </v-layout>
              </v-flex>
            </v-layout>
          </v-card-title>
          <v-layout row wrap>
            <v-flex xs9 sm11 md5>
              <v-data-table
                :headers="headers"
                :loading="isloading"
                :items="filteredReport"
                :pagination.sync="pagination"
                class="elevation-1 txt-title"
              >
                <v-progress-linear v-slot:progress color="blue" indeterminate></v-progress-linear>
                <template v-slot:items="props">
                  <td>{{ props.item.product.name }}</td>
                  <td>{{ props.item.count }}</td>
                  <td>{{ props.item.cost }}</td>
                  <td>{{ props.item.count*props.item.cost }}</td>
                </template>
                <template v-slot:footer>
                  <td :colspan="headers.length" v-if="check">
                    <b>รวมเป็นเงิน {{total}} บาท</b>
                  </td>
                </template>
              </v-data-table>
            </v-flex>
            <v-flex xs9 sm11 md7>
              <v-card>
                <v-card-title>
                  <h3 class="txt-title">กราฟวิเคราะห์จำนวนขาย(ชิ้น)</h3>
                </v-card-title>
                <apexchart type="donut" height="503" :options="chartOptions" :series="series" />
                <br />
              </v-card>
            </v-flex>
          </v-layout>
          <v-layout row wrap>
            <v-flex xs9 sm11 md12>
              <v-card>
                <v-card-title>
                  <v-layout align-center justify-space-between row fill-height>
                    <h3 class="txt-title mb-3">กราฟวิเคราะห์สินค้าขายดี 5 อันดับ</h3>
                    <v-flex xs12 sm5 md3 mt-2>
                      <v-select
                        :items="types"
                        v-model="type"
                        item-text="title"
                        label="ประเภทสินค้า"
                        solo
                      ></v-select>
                    </v-flex>
                  </v-layout>
                </v-card-title>
                <apexchart type="bar" height="350" :options="chartOptionsBar" :series="seriesBar" />
              </v-card>
            </v-flex>
          </v-layout>
        </v-container>
      </v-card>
    </v-flex>
  </v-layout>
</template>

<script>
import pdfsalereport from "./pdfReport/pdfsalereport";
var colors = [
  "#008FFB",
  "#00E396",
  "#FEB019",
  "#FF4560",
  "#775DD0",
  "#546E7A",
  "#26a69a",
  "#D10CE8"
];
export default {
  props: ["usernow"],
  data: () => ({
    check: false,
    is_day: true,
    dateTime: "",
    pagination: {
      rowsPerPage: 7
    },
    selectReport: { title: "รายงานประจำวัน", value: true },
    typeRep: [
      { title: "รายงานประจำวัน", value: true },
      {
        title: "รายงานประจำเดือน",
        value: false
      }
    ],
    date: new Date().toISOString().substr(0, 10),
    month: new Date().toISOString().substr(0, 7),
    type: "",
    types: [
      { title: "อาหาร/เครื่องดื่ม", value: "food" },
      {
        title: "เสื้อผ้า/เครื่องแต่งกาย/ชุดเครื่องนอน",
        value: "clothes"
      },
      {
        title: "อุปกรณ์เครื่องเขียน",
        value: "stationary"
      },
      {
        title: "อื่นๆ",
        value: "other"
      }
    ],
    modal: false,
    search: "",
    headers: [
      { text: "ชื่อ", sortable: false, value: "product.name" },
      { text: "จำนวน (ชิ้น)", sortable: false, value: "count" },
      {
        text: "ราคา (บาท)",
        sortable: false,
        value: "cost"
      },
      { text: "รวมเป็นเงิน (บาท)", sortable: false, value: "cost*count" }
    ],
    isloading: false,
    reports: [],
    price: [],
    forCountSumPrice: [],
    chartOptions: {
      labels: [
        "อาหาร/เครื่องดื่ม",
        "เสื้อผ้า/เครื่องแต่งกาย/ชุดเครื่องนอน",
        "อุปกรณ์เครื่องเขียน",
        "อื่นๆ"
      ],
      responsive: [
        {
          breakpoint: 480,
          options: {
            chart: {
              width: 200
            },
            legend: {
              position: "bottom"
            }
          }
        }
      ]
    }
  }),
  mounted() {
    this.getReportData();
  },
  methods: {
    getReportData() {
      this.isloading = true;
      axios
        .get("api/reportuser")
        .then(response => {
          this.reports = response.data;
          console.log(this.reports);
        })
        .then();
      this.isloading = false;
    },
    reportPdf() {
      if (this.is_day) {
        pdfsalereport.pdfMaker(this.filteredReport, this.total, this.date);
      }else{
        pdfsalereport.pdfMaker(this.filteredReport, this.total, this.month);
      }
    }
  },
  computed: {
    series: function() {
      var series = [0, 0, 0, 0];
      for (var i = 0; i < this.filteredReport.length; i++) {
        if (this.filteredReport[i].product.type == "food") {
          series[0] += this.filteredReport[i].count;
        } else if (this.filteredReport[i].product.type == "clothes") {
          series[1] += this.filteredReport[i].count;
        } else if (this.filteredReport[i].product.type == "stationary") {
          series[2] += this.filteredReport[i].count;
        } else {
          series[3] += this.filteredReport[i].count;
        }
      }
      return series;
    },
    seriesBar: function() {
      var seriesBar = [
        {
          name: "ยอดขาย(ชิ้น)",
          data: []
        }
      ];
      var i = 0;
      if (this.sortDataSet.length > 5) {
        for (var i = 0; i < 5; i++) {
          seriesBar[0].data[i] = parseInt(this.sortDataSet[i].count);
        }
      } else {
        for (var i = 0; i < this.sortDataSet.length; i++) {
          seriesBar[0].data[i] = parseInt(this.sortDataSet[i].count);
        }
      }
      console.log(seriesBar[0].data);
      return seriesBar;
    },
    chartOptionsBar: function() {
      var chartOptionsBar = {
        chart: {
          height: 350,
          type: "bar",
          events: {
            click: function(chart, w, e) {
              console.log(chart, w, e);
            }
          }
        },
        colors: colors,
        plotOptions: {
          bar: {
            columnWidth: "45%",
            distributed: true
          }
        },
        dataLabels: {
          enabled: true
        },
        xaxis: {
          categories: [],
          labels: {
            style: {
              colors: colors,
              fontSize: "14px"
            }
          }
        }
      };
      if (this.sortDataSet.length > 5) {
        for (var i = 0; i < 5; i++) {
          chartOptionsBar.xaxis.categories[i] = this.sortDataSet[
            i
          ].product.name;
        }
      } else {
        for (var i = 0; i < this.sortDataSet.length; i++) {
          chartOptionsBar.xaxis.categories[i] = this.sortDataSet[
            i
          ].product.name;
        }
      }
      return chartOptionsBar;
    },
    filteredReport: function() {
      var arrReport = [];
      if (this.is_day) {
        arrReport = this.reports.filter(report => {
          return report.created_at.substr(0, 10) == this.date;
        });
      } else {
        arrReport = this.reports.filter(report => {
          return report.created_at.substr(0, 7) == this.month;
        });
      }

      this.forCountSumPrice = arrReport;
      const result = [
        ...arrReport
          .reduce((mp, o) => {
            const key = JSON.stringify([o.cost, o.product]);
            if (!mp.has(key)) mp.set(key, { ...o, count: 0 });
            mp.get(key).count++;
            return mp;
          }, new Map())
          .values()
      ];
      this.check = true;
      return result;
    },
    total: function() {
      let total = [];
      Object.entries(this.forCountSumPrice).forEach(([key, val]) => {
        total.push(val.cost);
      });
      return total.reduce(function(total, num) {
        return total + num;
      }, 0);
    },
    sortDataSet: function() {
      var sortDataSet = this.filteredReport.sort((a, b) => b.count - a.count);
      return sortDataSet.filter(sort => {
        return sort.product.type.match(this.type);
      });
    },
    pageShow() {
      return "A3";
    }
    // date() {

    //   if (this.is_day) {
    //     return date;
    //   }
    //   else {
    //     return month;
    //   }
    // }
  }
};
</script>
<template>
  <div class="col-12">
    <div class="row">
      <div class="col-12">
        <ol class="breadcrumb">
          <li class="breadcrumb-item">
            <router-link :to="{name: 'Home'}">Dashboard</router-link>
          </li>
          <li class="breadcrumb-item active">Diapers</li>
        </ol>
      </div>
    </div>
    <div class="row" v-if="error">
      <div class="col-12">
        <div class="alert alert-danger">
          An error has occurred: {{ error }}.
          <br><a class="alert-link" href="" onclick="window.location.reload()">Click here to refresh</a>
        </div>
      </div>
    </div>
    <div class="row">
      <div class="col-12">
        <div class="card m-bl-30">
          <div class="card-body">
            <div class="col-12">
              <div class="row" style="margin-bottom: 15px">
                <div class="col-12">
                  <router-link :to="{name: 'NewDiaper'}">
                    <button class="btn btn-primary waves-effect waves-light float-right">
                        <i class="fa fa-plus"></i>
                        Add diaper
                    </button>
                  </router-link>
                </div> <!-- END COL -->
              </div> <!-- END ROW -->

              <div class="row">
                <div class="col-12">
                  <div class="table-rep-plugin">
                    <div class="table-responsive b-0" data-pattern="priority-columns">
                      <table id="tech-companies-1" class="table table-striped">
                        <thead>
                          <tr>
                            <th>Model</th>
                            <th data-priority="1">Description</th>
                            <th data-priority="2">Sizes</th>
                            <th data-priority="4">
                              <center>Action</center>
                            </th>
                          </tr>
                        </thead>
                        <tbody>
                          <tr
                            v-for="(row, index) in rows"
                            v-bind:item="row"
                            v-bind:index="index"
                            v-bind:key="row.id"
                          >
                            <td>{{ row.doc.model}}</td>
                            <td>{{ row.doc.description }}</td>
                            <td>{{ getSizes(index) }}</td>
                            <td>
                              <center>
                                <div>
                                  <b-dropdown id="ddown1" class="m-md-2">
                                    <template slot="button-content">
                                      <i class="ion-navicon-round"></i> Actions
                                    </template>
                                    <b-dropdown-item>
                                      <router-link :to="{name: 'BuyDiaper', params: {id: row.id}}">
                                        <i class="fas fa-cart-plus"></i>
                                        Buy diaper
                                      </router-link>
                                    </b-dropdown-item>
                                    <b-dropdown-item>
                                      <router-link :to="{name: 'ViewDiaper', params: {id: row.id}}">
                                        <i class="fa fa-eye"></i>
                                        More details
                                      </router-link>
                                    </b-dropdown-item>
                                    <!-- <b-dropdown-item> -->
                                      <!-- <router-link :to="{name: 'TeacherEdit', params: {id: row.id}}">
                                                                        <i class="ion-ios7-paper-outline"></i>
                                                                        Alterar dados do professor
                                      </router-link>-->
                                    <!-- </b-dropdown-item> -->
                                    <b-dropdown-divider></b-dropdown-divider>
                                    <b-dropdown-item v-on:click="archiveEntry(row.id, index)"> 
                                      <i class="fa fa-trash"></i> Archive diaper
                                    </b-dropdown-item>
                                  </b-dropdown>
                                </div>
                              </center>
                            </td>
                          </tr>
                          <!-- <pagination :data="rows" :show-disabled="true" :limit="3" @pagination-change-page="getRows"></pagination> -->
                        </tbody>
                      </table>
                    </div> <!-- END RESPONSIVE TABLE -->
                  </div> <!-- END TABLE REP -->
                </div> <!-- END COL -->
              </div> <!-- END ROW -->
            </div> <!-- END COL -->
          </div> <!-- END COL -->
        </div> <!-- END CARD -->
      </div> <!-- END COL -->
    </div> <!-- END ROW -->
  </div> <!-- END COL -->
</template>

<script>
import DiaperService from "@/services/DiaperService";

export default {
  name: "home",
  data() {
    return {
      rows: {},
      error: ''
    };
  },
  async beforeCreate() {},
  async mounted() {},
  async created() {
    try {
      this.rows = await DiaperService.getDiapers();
    } catch (err) {
      this.error = err;
    }
  },
  methods: {
    getSizes: function(index) {
      const data = this.rows[index].doc;
      const availableL = data.availableL;
      const availableM = data.availableM;
      const availableP = data.availableP;

      let sizes = [];
      if (availableL > 0) sizes.push("L");
      if (availableM > 0) sizes.push("M");
      if (availableP > 0) sizes.push("P");
      return sizes.join(" - ");
    },
    async archiveEntry(id, index) {
      await this.$swal({
        title: "Confirmation",
        text: "You really sure want archive this data?",
        type: "warning",
        showCancelButton: true,
        confirmButtonText: "Yeah, i sure",
        cancelButtonText: "No"
      }).then(async result => {
        if (result.value) {
          await DiaperService.archiveDiaper(id).then(async () => {
            await this.rows.splice(index, 1);
            await this.$swal(
              'Archived',
              `Diaper ${index} archived with success`,
              'success'
            );
          }).catch(async error => {
            this.error = error;
            await this.$swal(
              'Error',
              'An error has occurred',
              'error'
            );
          });
        } else if (result.dismiss === this.$swal.DismissReason.cancel) {
          await this.$swal("Canceled", "You cancelled the action. Your data is safe.", "error");
        }
      });
    }
  }
};
</script>

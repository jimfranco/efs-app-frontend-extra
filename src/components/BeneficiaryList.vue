<template>
  <v-main>
    <!-- Welcome title -->
    <v-container fluid>
      <v-row align="center" justify="center">
        <v-col cols="12" align="center" justify="center">
          <blockquote>
            Welcome {{validUserName}}!
            <footer>
              <small>
                <em>&mdash;Eagle Financial Services, your Midwest Financial Services Partner.</em>
              </small>
            </footer>
          </blockquote>
        </v-col>
        <v-col  cols="12" md="10" lg="10" align="center" justify="center">
          <v-alert v-if="showMsg === 'new'"
                   dismissible
                   :value="true"
                   type="success"
          >
            New beneficiary has been added.
          </v-alert>
          <v-alert v-if="showMsg === 'update'" dismissible
                   :value="true"
                   type="success"
          >
           Beneficiary information has been updated.
          </v-alert>
          <v-alert v-if="showMsg === 'deleted'" dismissible
                   :value="true"
                   type="success"
          >
            Selected beneficiary has been deleted.
          </v-alert>
        </v-col>
      </v-row>

      <!-- Data table -->
      <v-row align="center" justify="center">
        <v-col cols="12" md="10" v-resize="onResize">
            <v-data-table
              :headers="headers"
              :items="beneficiaries"
              class="elevation-1"
              style="max-height: 300px; overflow-y: auto"
              v-if="!isMobile"
            >
                    <template v-slot:item="props">
                      <tr>
                        <td align="left">{{ props.item.name }}</td>
                        <td align="center"><v-icon @click="updateBeneficiary(props.item)">mdi-pencil</v-icon></td>
                        <td align="center"><v-icon @click="deleteBeneficiary(props.item)">mdi-delete</v-icon></td>
                      </tr>  
                    </template>
              </v-data-table>
              <v-data-iterator 
                :items="beneficiaries"
                hide-default-footer
                v-else
              >
                <template v-slot:default="{ items, isExpanded, expand }">
                  <v-row>
                    <v-col
                      v-for="item in items"
                      :key="item.name"
                      cols="12"
                    >
                      <v-card>
                        <v-card-title class="pb-0 pt-0 pl-0">
                          <v-col cols="9" class="text-left body-2 text-truncate">{{item.name}}</v-col>
                          <v-col cols="3" class="text-center">
                            <v-card-actions>
                              <v-icon @click="updateBeneficiary(item)" class="small">mdi-pencil</v-icon>
                              <v-icon @click="deleteBeneficiary(item)" class="small">mdi-delete</v-icon>
                            </v-card-actions>
                          </v-col>
                        </v-card-title>
                        <v-divider></v-divider>
                      </v-card>
                    </v-col>
                  </v-row>
                </template>     
              </v-data-iterator>  
              <v-btn class="blue mt-4 white--text" @click="addNewBeneficiary">Add Beneficiary</v-btn>  
        </v-col>  
      </v-row>
    </v-container>  
  </v-main>
</template>


<script>

  import router from '../router';
  import {APIService} from '../http/APIService';
  const apiService = new APIService();

  export default {
    name: "BeneficiaryList",
    data: () => ({
      beneficiaries: [],
      validUserName: "Guest",
      beneficiarySize: 0,
      showMsg: '',
      isMobile: false,
      headers: [
        {text: 'Name', sortable: false, align: 'left',},
        {text: 'Update', sortable: false, align: 'left',},
        {text: 'Delete', sortable: false, align: 'left',}
      ],

    }),
    mounted() {
      this.getBeneficiaries();
      this.showMessages();
    },
    methods: {
      onResize() {
          if (window.innerWidth < 600)
            this.isMobile = true;
          else  
            this.isMobile = false;
        },
      showMessages(){
        console.log('Message: ' + this.$route.params.msg)
        if (this.$route.params.msg) {
          this.showMsg = this.$route.params.msg;
        }
      },
      getBeneficiaries() {
        apiService.getBeneficiaryList().then(response => {
          this.beneficiaries = response.data.data;
          this.beneficiarySize = this.beneficiaries.length;
          if (localStorage.getItem("isAuthenticates")
            && JSON.parse(localStorage.getItem("isAuthenticates")) === true) {
            this.validUserName = JSON.parse(localStorage.getItem("log_user"));
          }
        }).catch(error => {
          if (error.response.status === 401) {
            localStorage.removeItem('isAuthenticates');
            localStorage.removeItem('log_user');
            localStorage.removeItem('token');
            router.push("/auth");
          }
        });
      },
      addNewBeneficiary() {
        if (localStorage.getItem("isAuthenticates")
          && JSON.parse(localStorage.getItem("isAuthenticates")) === true) {
          router.push('/beneficiary-create');
        } else {
          router.push("/auth");
        }
      },
      updateBeneficiary(beneficiary) {
        router.push('/beneficiary-create/' + beneficiary.pk);
      },
      deleteBeneficiary(beneficiary) {
        apiService.deleteBeneficiary(beneficiary.pk).then(response => {
          if (response.status === 204) {
            router.push('/beneficiary-list/deleted/')
            this.getBeneficiaries();
            this.showMessages();
          }
        }).catch(error => {
          if (error.response.status === 401) {
            localStorage.removeItem('isAuthenticates');
            localStorage.removeItem('log_user');
            localStorage.removeItem('token');
            router.push("/auth");
          }
        });
      }
    }
  };
</script>

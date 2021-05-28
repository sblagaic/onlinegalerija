<template>
  <v-app>
    <div :class="{ 'light': !$store.getters.mode, 'dark': $store.getters.mode }">
      <div>
        <Header/>
      </div>


  <div class="gal_title text-center"
      style="cursor: pointer"
      @mouseover="descIn()" @mouseleave="descOut()"
      >
      Dobrodošli u online galeriju
  </div>
  
  <div class="gal_desc">
    {{description}}
  </div>

  <!-- ---------------- PRIJAVA ------------------------- -->
    <div v-if="this.$store.getters.sign_in_form" class="form" ref="enter_form">
      <!-- <div v-if="loading">
        <v-progress-circular
          :size="200"
          color="black"
          indeterminate
          style="margin-left: 30%;"
        ></v-progress-circular>
      </div> -->

      <div>
        <v-form
          ref="form"
          class="form_write"
          v-model="valid"
          lazy-validation
        >

          <v-text-field class="form_write"
            v-model="email_sign"
            :rules="emailRules"
            label="E-mail"
            required
          ></v-text-field>

          <v-text-field
            v-model="password_sign"
            :append-icon="show_sign ? 'mdi-eye' : 'mdi-eye-off'"
            :rules="[pass_rules.required, pass_rules.min]"
            :type="show_sign ? 'text' : 'password'"
            name="input-10-1"
            label="Lozinka"
            counter
            @click:append="show_sign = !show_sign"
            required
          ></v-text-field>

        <div class="form_buttons">
          <v-btn rounded 
            :disabled="!valid"
            color="rgba(1, 24, 12)"
            class="mr-4 white--text"
            @click="validate_s()"
          >
            Prijava
          </v-btn>

          <v-btn rounded
            color="rgb(33, 1, 1)"
            class="mr-4 white--text"
            @click="cancel()"
          >
            Natrag
          </v-btn>

        </div>
        </v-form>
      </div>
    </div>

    <!-- ------------------- REGISTRACIJA ------------------ -->
    <div v-if="this.$store.getters.register_form" class="form" ref="enter_form">

      <div>
        <v-form
          ref="form"
          v-model="valid"
          lazy-validation
        >
          <v-text-field
            v-model="name"
            :rules="[v => !!v || 'Potrebno je upisati ime']"
            label="Ime"
            required
          ></v-text-field>

          <v-text-field
            v-model="surname" 
            :rules="[v => !!v || 'Potrebno je upisati prezime']"
            label="Prezime"
            required
          ></v-text-field>

          <v-text-field
            v-model="email"
            :rules="emailRules"
            label="E-mail"
            required
          ></v-text-field>

          <v-text-field
            v-model="payPal"
            :rules="payPalRules"
            label="PayPal račun"
            required
          ></v-text-field>

          <v-text-field
            v-model="password_reg"
            :append-icon="show_reg ? 'mdi-eye' : 'mdi-eye-off'"
            :rules="[pass_rules.required, pass_rules.min]"
            :type="show_reg ? 'text' : 'password'"
            name="input-10-1"
            label="Lozinka"
            hint="Najmanje 8 znakova"
            counter
            @click:append="show_reg = !show_reg"
          ></v-text-field>

          <v-file-input v-if="artist_check"  
            type="file"
            v-model="pdf"
            :rules="[v => !!v || 'Potrebno je priložiti datoteku']"
            small-chips    
            accept="application/pdf"
            label="Priložite PDF datoteku"
          ></v-file-input>

          <v-alert
            :value="artist_check"
            color="black"
            dark
            transition="scale-transition"
          >
            <p style="font-weight: 100"> Ukoliko se želite registrirati kao umjetnik trebate priložiti svoj portfolio u obliku PDF datoteke. </p>
          </v-alert>
          
          <v-checkbox
            color="black"
            v-model="artist_check"
            label="Ja sam umjetnik"
          ></v-checkbox>

        <div class="form_buttons">
          <v-btn rounded
            :disabled="!valid"
            color="rgba(1, 24, 12)"
            class="mr-4 white--text"
            @click="validate_r()"
          >
            Registracija
          </v-btn>

          <v-btn rounded
            color="rgba(33, 1, 1)"
            class="mr-4 white--text"
            @click="cancel()"
          >
            Natrag
          </v-btn>

        </div>
        </v-form>
      </div>
    </div>

  <!-- --------------------- SLIKE -------------------------------- -->
    <v-dialog
      v-model="deleteExh"
      :retain-focus="false"
      persistent
      max-width="311"
    >
      <template v-slot:activator="{}">
        
      </template>
      <v-card>
        <v-card-title class="headline">
          Jeste li sigurni da želite izbrisati izložbu?
        </v-card-title>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="black" text @click="deleteExh = false">
            Ne
          </v-btn>
          <v-btn color="black" text @click="deleteExhibition(deleteExhIndex)">
            Da
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- ---------------- IZLOŽBE UŽIVO ---------------- -->
    <div class="exh_text"> 
      <p v-if="!filtered"> Naše izložbe </p> 
      <p v-else> Odabrane izložbe </p>
    </div>
    <v-row>
      <v-col
        v-for="n in noOfExhs" :key="n"
        class="d-flex child-flex"
        cols="12"
        sm="4"
      >
        <v-hover v-slot="{ hover }" open-delay="200">
          <v-card class="images"
            :elevation="hover ? 12 : 2"
            :class="{ 'on-hover': hover }"
          >
            <v-img
              :src="'data:image/jpg;base64,' + artSources[n - 1]"
              :lazy-src="'data:image/jpg;base64,' + artSources[n - 1]"
              aspect-ratio="1"
              class="grey lighten-2 img"
            >
            <v-expand-transition>
              <div
                v-if="hover"
                class="d-flex transition-fast-in-fast-out darken-2 v-card--reveal display-3"
                :class="!$store.getters.mode ? 'hover_light white--text' : 'hover_dark'"
                style="height: 50%;"
              >
              
                  <div class="exh_title"> <i><b v-if="artistsMultiple[n - 1] == false"> {{exhDescriptions[n - 1]["Artists"]}} </b><br> 
                    <p style="margin-top: -5%; font-weight: 100" 
                    v-bind:style= "artistsMultiple[n - 1] == false ? 'margin-top: -5%;' :'margin-top: -28%;'"  
                    > -{{exhDescriptions[n - 1]["Name"]}} </p></i> 
                  </div>
                
              </div>
            </v-expand-transition>
              <v-card-title class="align-end fill-height" primary-title>
                <v-tooltip bottom>
                  <template v-slot:activator="{ on: tooltip }">
                    <v-btn icon
                      v-bind="attrs"
                      v-on="{ ...tooltip, ...menu }"
                      :disabled="!$store.getters.logged_in"
                      @click="getExhibition(exhDescriptions[n - 1]['Name'], n)"
                      >
                      <v-icon :color="!$store.getters.mode ? 'white' : 'black'">mdi-door-open</v-icon>
                    </v-btn>
                  </template>
                  <span>Posjeti izložbu</span>
                </v-tooltip>
                <v-tooltip v-if="$store.getters.admin" bottom>
                  <template v-if="$store.getters.admin" v-slot:activator="{ on: tooltip }">
                    <v-btn v-if="$store.getters.admin" icon
                      v-bind="attrs"
                      v-on="{ ...tooltip, ...menu }"
                      :disabled="!$store.getters.logged_in"
                      @click="deleteExhIndex = n; deleteExh = true"
                      >
                      <v-icon :color="!$store.getters.mode ? 'white' : 'black'">mdi-close</v-icon>
                    </v-btn>
                  </template>
                  <span>Izbriši izložbu</span>
                </v-tooltip>
              </v-card-title>
            </v-img>
          </v-card>
        </v-hover>
      </v-col>
    </v-row>

    <!-- ---------- FILTERI ---------- -->
    <v-row justify="center">
      <v-dialog
        transition="dialog-bottom-transition"
        v-model="filters"
        persistent
        max-width="630"
      >
        <v-card>
          <v-card-title class="text-left justify-center py-6" style="font-size: 100%;">
            <div class="display-3" style="margin-right: auto; margin-bottom: 2%;">
              Prikaz
            </div>
            <div>
              Sortirajte izložbe prema odgovarajućem stilu, 
              pripadajućim umjetnicima i razdoblju trajanja
            </div>
          </v-card-title>

          <v-tabs
            v-model="tab"
            background-color="transparent"
            color="black"
            grow
          >
            <v-tab
              v-for="filter in filter_types"
              :key="filter"
            >
              {{ filter }}
            </v-tab>
          </v-tabs>

          <v-tabs-items v-model="tab">
            <v-tab-item>
              <v-card flat>
                <v-card-text class="black--text">Odaberite tehniku izložbi koje želite prikazati</v-card-text>
                <v-radio-group class="technique" v-model="selectedTech">
                  <v-radio
                    v-for="(teh, i) in techniques.length"
                    :key="teh"
                    :label="techniques[i]"
                    :value="teh"
                    color="black"
                  ></v-radio>
                  <v-radio
                    label="Poništite odabir"
                    @click="selectedTech = null"
                    :value="4"
                    color="black"
                  ></v-radio>
                </v-radio-group>
              </v-card>
            </v-tab-item>
            <v-tab-item>
              <v-card flat>
                <v-card-text class="black--text">Odaberite jednog ili više umjetnika čije izložbe želite prikazati</v-card-text>
                <v-autocomplete
                  v-model="selectedArtists"
                  :items="artists"
                  class="mx-4"
                  multiple
                  small-chips
                  deletable-chips
                  clearable
                  label="Umjetnici"
                  color="black"
                ></v-autocomplete>
              </v-card>
            </v-tab-item>
            <v-tab-item>
              <v-card flat>
                <v-card-text class="black--text">Odaberite dva datuma između kojih će prikazane izložbe biti otvorene</v-card-text>
                <div class="calendar">
                  <v-date-picker
                    v-model="exh_dates"
                    scrollable
                    range
                    locale="hr"
                    :elevation="1"
                    :first-day-of-week="1"
                    selected-items-text="Željeni raspon"
                    color="black"
                  ></v-date-picker>
                </div>
              </v-card>
            </v-tab-item>
          </v-tabs-items>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn
              color="black"
              text
              @click="filters = false"
            >
              Natrag
            </v-btn>
            <v-btn
              color="black"
              text
              @click="applyFilters()"
            >
              Primijeni
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-row>
    <v-btn fab 
          :dark="!$store.getters.mode" 
          :color="$store.getters.mode ? 'white' : 'rgb(0, 0, 0, 0.9)'"
          fixed right bottom
          @click="filters = true"
          >
      <v-icon>mdi-sort</v-icon>
    </v-btn>

    <v-snackbar
      timeout="3000"
      :value="snackReg"
      outlined
      multi-line
    >
      <p style="text-align: center; margin-bottom: -1%; font-size: 16px;">      
      Registracija uspješna. Molimo <br>
      prijavite se ukoliko želite nastaviti.</p>
    </v-snackbar>    

    <v-snackbar
      v-model="exhOpened"
      multi-line
      color="rgb(33, 1, 1)"
    >
      Izložba se otvara {{snackDate[2]}}.{{snackDate[1]}}.{{snackDate[0]}}.

      <template v-slot:action="{ attrs }">
        <v-btn
          color="white"
          text
          v-bind="attrs"
          @click="exhOpened = false"
        >
          Close
        </v-btn>
      </template>
    </v-snackbar>

  </div>

  </v-app>
</template>

<script>
import HelloWorld from '../components/HelloWorld';
import Header from '@/components/Header'
import axios from 'axios'

export default {
  name: 'App',

  components: {
    HelloWorld,
    Header
  },

  data: () => {
    return {
      description: '',
      loading: false,
      snackReg: false,
      sign_att: false,
      register_att: false,
      timer: null,
      form: false,
      sign_in_form: false,
      register_form: false,
      enter_exh: false,
      artist_check: false,

      pdf: 'empty_file',
      valid: true,
      name: '',
      surname: '',
      payPal: '',
      password_sign: '',
      password_reg: '',
      nameRules: [
        v => !!v || 'Name is required',
      ],
      email: '',
      email_sign: '',
      emailRules: [
        v => !!v || 'Potrebno je upisati E-mail',
        v => /.+@.+\..+/.test(v) || 'E-mail mora biti valjan',
      ],
      payPalRules: [
        v => !!v || 'Potrebno je upisati PayPal račun',
        v => /.+@.+\..+/.test(v) || 'Paypal račun mora biti valjan',
      ],
      select: null,
      checkbox: false,
      show_sign: false,
      show_reg: false,
      pass_rules: {
          required: value => !!value || 'Potrebno je upisati lozinku.',
          min: v => v.length >= 8 || 'Minimalno 8 znakova',
          emailMatch: () => (`The email and password you entered don't match`),
        },
      is_admin: false,
      exh_dates: [],
      filters: false,
      tab: null,
      filter_types: ['Stil', 'Umjetnici', 'Datum'],
      radioGroup: 0,
      
      artists: [],
      artistsMultiple: [],
      artistsMultipleOrigi: [],
      techniques: [],    
      selectedArtists: [],
      selectedTech: null,
      hasSaved: false,
      model: null,

      collections: null,
      exhibits: null,
      exhibitsFiltered: [],
      exhDescriptions: [],
      exhDescriptionsOrigi: [],
      artSources: [],
      artSourcesOrigi: [],
      noOfExhs: 0,
      noOfExhsOrigi: 0, 
      imgMapVals: null,
      imagesExh: [],
      imagesInfo: [],
      filtered: false,
      exhOpened: false,
      exhBeginDate: null,
      snackDate: '2020-01-01',
      deleteExh: false,
      deleteExhIndex: 0
    }
  },

  mounted() {
        this.setUser()

        this.sign_in_form = this.$store.getters.sign_in_form
        var logged = (sessionStorage.getItem('logged_in') === 'true');
        this.$store.commit('show_tool', logged ? true : false)

        this.getExhibitionSingles()
    },

  methods: {
    setUser() {
      if (sessionStorage.getItem('userType') === '1') {
      this.$store.commit('log_admin', true)
      this.$store.commit('log_artist', false)
      } else if (sessionStorage.getItem('userType') === '2') {
        this.$store.commit('log_admin', false)
        this.$store.commit('log_artist', true)
      } else {
        this.$store.commit('log_admin', false)
        this.$store.commit('log_artist', false)
      }
    },

    descIn() {
      this.timer = setTimeout(() => { 
        this.description = `Online galerija osnovana je 2020. godine s ciljem promicanja kulture i umjetnosti. 
      Plod mladih i britkih umova, online galerija stremi podizanju svijesti i javnog mnijenja o umjetnosti te širenju iste među mladima.`;
      }, 450);
    },

    descOut() {
      setTimeout(() => { 
        this.description = '';
        clearTimeout(this.timer);
      }, 300);
    },

    sign_in() {
      this.$store.commit('sign_in', true)
      this.$store.commit('register', false)
    },

    register() {
      this.$store.commit('register', true)
      this.$store.commit('sign_in', false)
    },

    sign_out() {
      this.$store.commit('show_tool', false)
      this.$store.commit('sign_in', false)
      this.$store.commit('register', false)
      this.$store.dispatch('logout')
        .then(() => {
          this.sign_out_success()
        })
    },

    sign_out_success() {
      this.$store.commit('show_tool', false)
      this.$store.commit('sign_in', false)
      this.$store.commit('register', false)
      this.$router.push('/')
    },

    validate_s() {
        this.$refs.form.validate()
        this.loading = true;
        let email = this.email_sign
        let password = this.password_sign
        this.$store.dispatch('login', { email, password })
       .then(() => this.sign_success())
       .catch(err => console.log(err))
    },

    sign_success() {
        if (this.valid) {
          this.$store.commit('show_tool', true)
          this.$store.commit('sign_in', false)
          this.$store.commit('register', false)
        }

        if (this.$store.getters.logged_in) 
            this.enter_exh = true;

        this.$store.commit('sign_in', false)
        this.$store.commit('register', false)
        this.loading = false
        this.$router.push('/')
    },

    validate_r() {
        let data = {
          name: this.name,
          surname: this.surname,
          email: this.email,
          password: this.password_reg,
          paypalMail: this.payPal,
          flag: this.pdf == 'empty_file' ? false : true
        }
        
        this.loading = true
        this.$store.commit('set_user', data)
        data = JSON.stringify(data)
        let formData = new FormData()
        formData.append('file', this.pdf)
        formData.append('json', new Blob([
            data
        ], {
            type: "application/json"
        }))

        this.$store.dispatch('register', formData)
       .then(() => this.validate_success())
       .catch(err => console.log(err))
    },

    validate_success() {
      this.$refs.form.validate()
      this.snackReg = true
      this.$store.commit('sign_in', false)
      this.$store.commit('register', false)
      this.loading = false
      this.$router.push('/')
        
    },

    cancel() {
      this.$refs.form.reset()
      this.$store.commit('sign_in', false)
      this.$store.commit('register', false)
    },

    getCollections() {
      axios({url: `${process.env.VUE_APP_BACKEND_URI}/artist/getCollections`, 
            headers: {
              'Authorization':  `Bearer ${sessionStorage.getItem('token')}`
            },
            params: {
              'type' : 'all'
            },
            method: 'GET'
      })
      .then((response) => {
        this.collections = response.data;
      })
      .catch(err => {
          console.log(err)
      });
    },

    getExhibition(exhName, n) {
      var currentDate = new Date()
      this.exhBeginDate = this.exhDescriptions[n - 1]['BeginDate']
      let tmp = this.exhBeginDate.split("T")
      this.snackDate = tmp[0].split("-")
      if (currentDate < this.getvalidDate(this.exhBeginDate)) {
        this.exhOpened = true
      } else {
        axios({url: `${process.env.VUE_APP_BACKEND_URI}/visitor/getExhibition`, 
              headers: {
                'Authorization':  `Bearer ${sessionStorage.getItem('token')}`
              },
              params: {
                'exName' : exhName
              },
              method: 'GET'
        })
        .then((response) => {
          localStorage.setItem('exhibition', JSON.stringify(response.data))
          this.$router.push('/izlozba')
        })
        .catch(err => {
            console.log(err)
        });
      }
    },
    
    getExhibitionSingles() {
      axios({url: `${process.env.VUE_APP_BACKEND_URI}/visitor/getExhibitionSingles`, method: 'GET'})
      .then((response) => {
        this.exhibits = response.data;
        let i = 0
        let tech = 0
        
        for (let [description, value] of Object.entries(this.exhibits)) {
            this.exhDescriptions[this.noOfExhs] = JSON.parse(description)
            this.artSources[this.noOfExhs] = value
            this.noOfExhs++
            for (let [key, info] of Object.entries(JSON.parse(description))) {
                if (key == 'Artists') {
                  this.artistsMultiple[i] = false
                  if (info.includes(",")) {
                      this.artistsMultiple[i] = true
                      var tmp = info.split(",")
                      for (let j = 0; j < tmp.length; j++) {
                          this.artists[i] = tmp[j]
                          i++
                      }
                  } else {
                    this.artists[i] = info
                    i++
                  }
                } else if (key == 'Style') {
                  info = info.toLowerCase()
                  info = info.charAt(0).toUpperCase() + info.slice(1)
                  info = info.replace(/_/g, ' ')
                  this.techniques[tech] = info
                  tech++
                }
            }
        }

        this.noOfExhsOrigi = this.noOfExhs
        this.artSourcesOrigi = this.artSources
        this.exhDescriptionsOrigi = this.exhDescriptions
        this.artistsMultipleOrigi = this.artistsMultiple

        this.artists = [...new Set(this.artists)];
        this.techniques = [...new Set(this.techniques)];
        this.filtered = false
        
      })
      .catch(err => {
          console.log(err)
      });
    },

    applyFilters() {
      this.resetExh()

      if (this.selectedTech == null && this.selectedArtists.length == 0 && this.exh_dates.length < 2) {
          this.revertExhOrigi()
      } else {
        let techFilter = false
        let artistFilter = false
        let dateFilter = false

        for (let [description, value] of Object.entries(this.exhibits)) {
            let tech = null
            for (let [key, info] of Object.entries(JSON.parse(description))) {
                if (key == 'Style') {                    
                    tech = info
                    tech = tech.toLowerCase()
                    tech = tech.charAt(0).toUpperCase() + tech.slice(1)
                    tech = tech.replace(/_/g, ' ')
                    let selTech = ''
                    if (this.selectedTech != null) 
                        selTech = this.techniques[this.selectedTech - 1].toString()

                    techFilter = selTech == tech.toString() || this.selectedTech === null

                } else if (key == 'Artists') {
                    this.artistsMultiple[this.noOfExhs] = false
                    if (info.includes(",")) {
                        this.artistsMultiple[this.noOfExhs] = true
                        var tmp = info.split(",")
                        this.artistsFiltered = tmp
                    } else {
                      this.artistsFiltered = info
                    }
                    if (this.selectedArtists.length > 0) {
                      if (Array.isArray(this.artistsFiltered)) {
                          for (let i = 0; i < this.artistsFiltered.length; i++)
                              artistFilter = this.filteredArtistExists(i, false)

                      } else {
                        artistFilter = this.filteredArtistExists(0, true)
                      }
                    } else {
                      artistFilter = true
                    }

                } else if (key == 'BeginDate') {
                  dateFilter = this.isBetweenDates(this.exh_dates[0], this.exh_dates[1], info)
                  if (this.exh_dates.length < 2)
                    dateFilter = true
                }
            }
            if (techFilter && artistFilter && dateFilter) {
              this.artSources[this.noOfExhs] = value
              this.exhDescriptions[this.noOfExhs] = JSON.parse(description)
              this.noOfExhs++
            } 
        }
      }
      this.filtered = true
      this.filters = false
    },

    deleteExhibition(n) {
      axios({url: `${process.env.VUE_APP_BACKEND_URI}/admin/closeExhibition`, 
            headers: {
              'Authorization':  `Bearer ${sessionStorage.getItem('token')}`,
              'Content-Type': 'application/json'
            },
            params: {
              'exhName' : this.exhDescriptions[n - 1]["Name"]
            }, 
            method: 'POST'
      })
      .then((response) => {
       this.deleteExh = false
       window.location.reload();
      })
      .catch(err => {
          console.log(err)
      });
    },

    setFilters() {
      this.selectedTech = sessionStorage.getItem('selectedTech')
      this.selectedArtists = sessionStorage.getItem('selectedArtists')
    },

    resetExh() {
      sessionStorage.setItem('filtered', true)
      this.noOfExhs = 0
      this.artSources = []
      this.exhDescriptions = []
      this.artistsMultiple = []
    },
    
    revertExhOrigi() {
      sessionStorage.setItem('filtered', false)
      this.noOfExhs = this.noOfExhsOrigi
      this.artSources = this.artSourcesOrigi
      this.exhDescriptions = this.exhDescriptionsOrigi
      this.artistsMultiple = this.artistsMultipleOrigi
    },

    filteredArtistExists(i, single) {
      if (this.selectedArtists.indexOf(single ? this.artistsFiltered : this.artistsFiltered[i]) != -1 
          || this.selectedArtists.length == 0) {
        return true
      } else {
        return false
      }
    },

    getvalidDate :function(d) { 
      return new Date(d) 
    },

    isBetweenDates(fromDate, toDate, givenDate) {
      return this.getvalidDate(givenDate) <= this.getvalidDate(toDate) 
              && this.getvalidDate(givenDate) >= this.getvalidDate(fromDate);
    },

    isOpened(openingDate) {
      var currentDate = new Date()
      return currentDate <= this.getvalidDate(openingDate) 
    }
    
  },


  computed: {
      dateRangeText () {
        return this.exh_dates.join(' ~ ')
      }
  },

  watch: {
      search (val) {
        val && val !== this.select && this.querySelections(val)
      }
  },
  
};
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Work+Sans:wght@100;400&display=swap');

.light {
  background-color: white;
  color: black;
}

.dark {
  background-color: black;
  color: white;
}

.prijava {
  margin-top: 7px;
}

.form {
  align-content: center;
  margin: auto;
  margin-top: -3%;
  max-width: 50%;
  padding: 2%;
  border: 1px solid black;
  border-radius: 50px;
  background-color: white;
}

.form::placeholder {
  color: black;
}

.form_write::placeholder {
  color: black;
}

.form_buttons {
  align-content: center;
  margin-right: auto;
  margin-left: auto;
}

.gal_title {
  align-content: center;
  justify-content: center;
  font-size: 100px;
  margin-top: 10%;
  font-family: 'Work Sans', sans-serif;
}

.exh_text {
  font-size: 50px;
  font-family:  'Work Sans', sans-serif;
  margin-left: 2%;
  margin-top: 5%;
}

.v-card--reveal {
  align-items: center;
  bottom: 0;
  justify-content: center;
  opacity: 0.8;
  position: absolute;
  width: 100%;
}

.hover_light {
  background-color: black;
}

.hover_dark {
  background-color: rgba(255, 255, 255, 1);
}

.images {
  align-content: right;
  justify-content: center;
  margin: auto;
  margin-top: 5%;
  margin-bottom: 5%;
  height: 0%;
  width: 95%;
  border-radius: 50px;
}

.img_hov {
  border-radius: 50px;
}

.img {
  margin: 5%;
}

.izl {
  font-size: 60%;
  font-family: 'Work Sans', sans-serif;
}

.exh_title {
  text-align: left;
  margin-top: -24%;
  margin-right: auto;
  margin-left: 2%;
  font-size: 60%;
}

.exh_name {
  font-weight: 100;
}

.technique {
  margin-left: 2%;
}

.calendar {
  text-align: center;
  margin-top: 4%;
  margin-bottom: 1%;
}

.gal_desc {
  margin: auto;
  text-align: center;
  margin-top: 2%;
  margin-bottom: 8%;
  max-width: 60%;
  font-size: 26px;
}

.gal_title {
  align-content: center;
  justify-content: center;
}

</style>

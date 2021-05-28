<template>
  <div id="app">
    <v-card class="overflow-hidden">
      <v-app-bar
        fixed
        color="#fcb69f"
        dark
        shrink-on-scroll
        src="@/assets/coland.jpg"
        
      >
        <template v-slot:img="{ props }">
          <v-img v-if="!$store.getters.mode"
            v-bind="props"
            gradient="to top right,  rgba(0, 0, 0, 0.8),  rgba(0, 0, 0, 0.6)"
          ></v-img>

          <v-img v-else
            v-bind="props"
            gradient="to top right,  rgba(255, 255, 255, 0.5),  rgba(255, 255, 255, 0.3)"
          ></v-img>
        </template>

        <v-toolbar-title @click="home()"
          :class="{'tool_title': !$store.getters.mode, 'tool_title black--text': $store.getters.mode}"
          style="cursor: pointer;"
          >
          onlinegalerija
        </v-toolbar-title>

        <v-spacer></v-spacer>

        <div v-if="!this.$store.getters.logged_in">
          <v-btn text rounded @click="sign_in()" :class="{'black--text': $store.getters.mode}">prijava</v-btn>
          <v-btn text rounded @click="register()" :class="{'black--text': $store.getters.mode}">registracija</v-btn>
          <v-btn icon @click="mode()">
            <v-icon :color="!$store.getters.mode ? 'white' : 'black'">mdi-coach-lamp</v-icon>
          </v-btn>
        </div>

        
        <div v-else>
          <v-btn icon @click="mode()">
            <v-icon :color="!$store.getters.mode ? 'white' : 'black'">mdi-coach-lamp</v-icon>
          </v-btn>
          <v-btn icon @click="home()">
            <v-icon :color="!$store.getters.mode ? 'white' : 'black'">mdi-home-roof</v-icon>
          </v-btn>

          <v-menu rounded="b-xl">
            <template v-slot:activator="{ on: menu, attrs }">
              <v-tooltip bottom>
                <template v-slot:activator="{ on: tooltip }">
                  <v-btn icon
                    v-bind="attrs"
                    v-on="{ ...tooltip, ...menu }"
                  >
                    <v-icon :color="!$store.getters.mode ? 'white' : 'black'">mdi-menu</v-icon>
                  </v-btn>
                </template>
                <span>Izbornik</span>
              </v-tooltip>
            </template>

            <v-list>
              <v-list-item to="/transakcije" link> Transakcije <v-spacer></v-spacer> <v-icon>mdi-cash-multiple</v-icon> </v-list-item>
              <v-list-item v-if="$store.getters.artist" to="/moj_profil" link > Moj profil <v-spacer></v-spacer> <v-icon>mdi-account-details</v-icon> </v-list-item>
              <v-list-item v-if="$store.getters.artist || $store.getters.admin" to="/natjecaji" link> Natječaji <v-spacer></v-spacer> <v-icon>mdi-medal-outline</v-icon> </v-list-item>
              <v-list-item link @click="sign_out()"> Odjava <v-spacer></v-spacer> <v-icon>mdi-exit-to-app</v-icon> </v-list-item>
            </v-list>
          </v-menu>
        </div>
      </v-app-bar>   
    </v-card>

    <v-snackbar
      timeout="3000"
      :value="snackOut"
      outlined
      multi-line
    >
      <p style="text-align: center; margin-bottom: -1%; font-size: 16px;">      
        Uspješno ste se odjavili.
      </p>
    </v-snackbar>

    <router-view/>
  </div>
  
</template>

<script>
import axios from 'axios'

export default {
  name: 'Header',

  data: () => {
    return {
      snackOut: false,
      form: false,
      valid: true,
    }
  },

  computed: {

  },

  methods: {
    home() {
      this.$router.push('/')
    },

    snackbar() {
      setTimeout(() => { 
        this.snackOut = false;        
      }, 3000);
    },

    sign_in() {
      this.$store.commit('sign_in', true)
      this.$store.commit('register', false)
      this.$vuetify.goTo(180)
    },

    register() {
      this.$store.commit('register', true)
      this.$store.commit('sign_in', false)
      this.$vuetify.goTo(380)
    },

    sign_out() {
      this.$store.dispatch('logout')
        .then(() => this.sign_out_success())
        .catch(err => console.log(err))
    },

    sign_out_success() {
      this.$store.commit('show_tool', false)
      this.$store.commit('sign_in', false)
      this.$store.commit('register', false)
      this.snackOut = true
      this.$router.push('/')
    },

    validate() {
        this.$refs.form.validate()
        if (this.valid) {
          this.form = true;
          this.$store.commit('show_tool', true)
        }
    },

    cancel() {
      this.$refs.form.reset()
    },

    home() {
      this.$router.push('/')
    },

    mode() {
      this.$store.getters.mode ? this.$store.commit('theme', false) : this.$store.commit('theme', true)
    },

    create_admin() {
      axios({url: `${process.env.VUE_APP_BACKEND_URI}/visitor/test`, method: 'POST'})
      .then((response) => {
        
      })
      .catch(err => {
          console.log(err)
      });
    },

    change_admin() {
      if(this.$store.getters.admin) {
          this.$store.commit('log_admin', false);
        } else {
          this.$store.commit('log_admin', true);
        }
    }
  }
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Work+Sans:wght@100;400&display=swap');

#app {
  font-family: 'Work Sans', sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.tool_title {
  font-weight: 200;
  color: "{$store.getters.mode ? black : white}";
}

.prijava {
  font-weight: 100;
}

.btn {
    text-transform: none !important;
}

.home {
    text-transform: none !important;
    font-size: 28px !important;
    font-family: 'Work Sans', sans-serif !important;
    -webkit-font-smoothing: antialiased !important;
    -moz-osx-font-smoothing: grayscale !important;
}

</style>

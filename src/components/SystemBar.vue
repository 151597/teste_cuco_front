<template>
    <v-card>
        <v-navigation-drawer
            v-model="drawer"
            app
            class="navigation"
            width="120"
            style="border-radius: unset;"
        >
            <v-list-item class="d-flex justify-center mb-2 mt-2">
                <router-link to="/">
                    <v-img
                        max-height="35"
                        max-width="35"
                        :src="require('@/assets/logo.png')">
                    </v-img>
                </router-link>
            </v-list-item>
            <!-- MENUS -->
            <v-list dense nav>
                <Cliente />
            </v-list>
        </v-navigation-drawer>

        <v-app-bar app class="navBar" white style="border-radius: unset;">

        <v-app-bar-nav-icon white @click="drawer = !drawer"></v-app-bar-nav-icon>

        <v-toolbar-title navFontSize style="padding-left: 0px">Teste Cuco</v-toolbar-title>
        <v-spacer></v-spacer>

        <v-toolbar-title navFontSize>Olá, Mock</v-toolbar-title>
        <v-menu
          bottom
          left
        >
          <template v-slot:activator="{ on, attrs }">
            <v-btn
              icon
              v-bind="attrs"
              v-on="on"
            >
              <v-icon>mdi-account-circle</v-icon>
            </v-btn>
          </template>

          <v-list>
            <v-list-item>
              <v-list-item-title><a @click.prevent="logout">Logout</a></v-list-item-title>
            </v-list-item>
          </v-list>
        </v-menu>
        </v-app-bar>
    </v-card>
</template>

<script>
import Cliente from './menus/Cliente'
export default {
    name: 'systemBar',
    components: {
        Cliente
    },
    data(){
        return {
            drawer: true,
            mini: true
        }
    }

}
</script>

<style>
    .navBar {
        background: linear-gradient(90deg, rgb(2, 115, 209) 0%, rgb(82, 216, 131) 100%) 0% 0% no-repeat padding-box padding-box transparent;
        opacity: 1;
    }
    .navigation {
        background: transparent linear-gradient(180deg, #0273D1 0%, #52D883 100%) 0% 0% no-repeat padding-box;
    }
    [white] {
        color: white !important;
    }
    [iconSize] {
        font-size: 18px !important;
    }
    hr {
        background-color:white;
        margin-top: 0.5rem !important;
        margin-left: 18px;
        margin-right: 18px;
    }
    [navFontSize] {
        font-size: 15px !important;
    }
    .v-navigation-drawer__content::-webkit-scrollbar {
        width: 5px !important;
        margin: 0;
    }
    .v-navigation-drawer__content::-webkit-scrollbar-track {
        background: radial-gradient(circle at 14px 18px, white 5px, transparent 7px) no-repeat, radial-gradient(circle at 14px calc(100% - 18px), white 5px, transparent 7px) no-repeat, linear-gradient(white, white) center no-repeat;
        box-shadow: inset 0 0 10px 12px white;
        background-size:  auto, auto, calc(100% - 16px) calc(100% - 40px);
        margin: 0;
    }
    .v-navigation-drawer__content::-webkit-scrollbar-thumb {
        background-color:rgb(5, 127, 226);
        border: 8px solid transparent;
        border-radius: 15px;
    }
    .theme--light.v-navigation-drawer:not(.v-navigation-drawer--floating) .v-navigation-drawer__border{
        background-color: transparent !important;
    }
</style>



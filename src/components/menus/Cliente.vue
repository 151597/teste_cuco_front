<template>
    <div>
        <router-link to="/clientes" style="color:white;text-decoration: none;">
            <div>
                <v-icon white large iconSize class="d-flex">mdi-account-circle</v-icon>
                <v-list-item-title class="d-flex justify-center" style="font-size: 13px;">Clientes</v-list-item-title>
                <hr>
            </div>
        </router-link>
    </div>
</template>

<script>
export default {
    name: 'Cliente'
}
</script>

<style>
    [iconSize] {
        font-size: 18px !important;
    }
</style>

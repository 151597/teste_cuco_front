<template>
    <div>
        <router-link to="/usuario" style="color:white;text-decoration: none;">
            <div>
                <v-icon white large iconSize class="d-flex">mdi-account-circle</v-icon>
                <v-list-item-title class="d-flex justify-center" style="font-size: 13px;">Usuario</v-list-item-title>
                <hr>
            </div>
        </router-link>
    </div>
</template>

<script>
export default {
    name: 'Usuario'
}
</script>

<style>
    [iconSize] {
        font-size: 18px !important;
    }
</style>

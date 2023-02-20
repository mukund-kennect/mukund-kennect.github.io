<template>
      <v-toolbar  color="#272727" class="navbar-custom">
        <v-icon end icon="mdi-github" class="ml-4"></v-icon>

        <v-toolbar-title class="font-weight-medium">Github Tracker - Mukund</v-toolbar-title>
        
</v-toolbar>
    <v-container fluid class="mt-5">
      
        <v-row class="mx-2">
            <v-col cols="12" md="4" sm="6">
                <v-text-field prepend-icon="mdi-account-badge" :rules="[
                    v => !!v || 'username feild is required'
                ]" v-model="githubUsername" label="Github Username" variant="underlined"></v-text-field>
            </v-col>

            <v-col cols="12" md="4" sm="6">
                <v-text-field prepend-icon="mdi-source-repository" :rules="[
                    v => !!v || 'Repository feild is required'
                ]" v-model="githubRepoName" label="Github Repository Name" hint="Make sure repository is public"
                    variant="underlined"></v-text-field>
            </v-col>
            <v-col cols="12" md="4" sm="6" class="mt-md-2">

                <v-btn :disabled="!isDataValid" color="white" @click="showRepoData">
                    {{buttonName  }}
                    <v-icon end icon="mdi-arrow-right-thick"></v-icon>
                </v-btn>
            </v-col>

        </v-row>
        <div v-if="isReadyToShow">
            <RepoData :username="githubUsername" :reponame="githubRepoName"></RepoData>
        </div>
    </v-container>


</template>

<script>
export default {
    data: () => ({
        githubUsername: '',
        githubRepoName: '',
        buttonName:'Find',
        isReadyToShow: false,
    }),

    computed: {
        isDataValid: function () {
            if (this.githubRepoName.length > 0 && this.githubUsername.length > 0) {
                return true;
            }
            return false;
        }
    },
    methods: {
        showRepoData() {
            if(!this.isReadyToShow){
                this.isReadyToShow=true;
                this.buttonName='Clear';
            }else{
                this.isReadyToShow=false;
                this.buttonName='Find';
            }
            console.log('button clicked');
        }
    }

}
</script>
<style>
.navbar-custom{
position: sticky;
z-index: 2;
top:0;

}
</style>

<script>
import { ref, computed } from 'vue'
import { useRouter } from 'vue-router'
import store from '../store'

export default {
  setup(props, context) {
    const accessToken = store?.tokenInfo?.access_token
    const authenticated = computed( ()=> store.authenticated )
    const user = computed( ()=> store.user )

    const router = useRouter()

    function confirmLogout() {
          const confirmed = confirm('Are you sure you want to logout?')
          if(confirmed) router.push({name: 'logout'})
    }
    
    return {
      authenticated,user,
      confirmLogout
    }
  }
}
</script>

<template>
 <nav class="navbar navbar-expand-lg navbar-light bg-light">
  <div class="container-fluid">
    <ul class="nav">
      <li class="nav-item"><router-link  :to="{name:'home'}" v-slot="{isExactActive, href}" custom>
        <a class="nav-link" :href="href" :class="isExactActive ? 'active' : ''">Home</a>
      </router-link></li>
      <li class="nav-item"><router-link class="nav-link" :to="{name:'listItems',params:{collection:'opera'}}">Opere</router-link></li>
      <li class="nav-item"><router-link class="nav-link" :to="{name:'listItems',params:{collection:'autore'}}">Autori</router-link></li>
      <li class="nav-item"><router-link class="nav-link" :to="{name:'notes'}">Notes</router-link></li>
    </ul>

    
    <template v-if="authenticated">
      <ul class="nav">
        <li class="nav-item dropdown">
            <a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown" aria-expanded="false">
              <span>
                <font-awesome-icon icon="fa-solid fa-user" fixed-width/>
                <span class="ms-2">{{user.first_name}}</span>
            </span>
            </a>
            <ul class="dropdown-menu dropdown-menu-end">
              <!-- <li><a class="dropdown-item" href="#">Another action</a></li>
              <li>
                <hr class="dropdown-divider">
              </li> -->
              <li><a class="dropdown-item" href="#" @click.prevent="confirmLogout">
                <font-awesome-icon icon="fa-regular fa-circle-xmark" fixed-width/>
                <span class="ms-2">Logout</span>
              </a></li>
            </ul>
          </li>
      </ul>
    </template>
    <template v-else>
      <ul class="nav">
        <li class="nav-item"><router-link class="nav-link" :to="{name:'login'}">
          <button class="btn btn-sm btn-primary">
            <font-awesome-icon icon="fa-solid fa-right-to-bracket" fixed-width/>
            <span>Login</span>
          </button>
        </router-link></li>
      </ul>
    </template>
  </div>
</nav>
</template>

<style scoped>
.active {
  font-weight: bold;
}
</style>

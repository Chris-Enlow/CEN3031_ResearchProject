<script setup>
import { supabase } from '../supabase'
import { onMounted, ref, toRefs } from 'vue'
import Roller from './Roller.vue'

const props = defineProps(['session'])
const { session } = toRefs(props)

const loading = ref(true)
const points = ref(0)

onMounted(() => {
  getProfile()
})

async function getProfile() {
  try {
    loading.value = true
    const { user } = session.value

    const { data, error, status } = await supabase
      .from('profiles')
      .select(`points`)
      .eq('id', user.id)
      .single()

    if (error && status !== 406) throw error

    if (data) {
      points.value = data.points
    }

  } catch (error) {
    alert(error.message)
  } finally {
    loading.value = false
  }
}

async function updateProfile() {
  try {
    loading.value = true
    const { user } = session.value

    const updates = {
      id: user.id,
      email: user.email,
      points: points.value,
    }

    const { error } = await supabase.from('profiles').upsert(updates)

    if (error) throw error
  } catch (error) {
    alert(error.message)
  } finally {
    loading.value = false
  }
}


async function signOut() {
  try {
    loading.value = true
    const { error } = await supabase.auth.signOut()
    if (error) throw error
  } catch (error) {
    alert(error.message)
  } finally {
    loading.value = false
  }
}

async function roller() {
  try {
    loading.value = true
    const { user } = session.value

    if(points.value >= 300)
      points.value-=300

    const updates = {
      id: user.id,
      email: user.email,
      points: points.value,
    }

    const { error } = await supabase.from('profiles').upsert(updates)

    if (error) throw error
  } catch (error) {
    alert(error.message)
  } finally {
    loading.value = false
  }
}
</script>

<template>
  <form class="form-widget" @submit.prevent="updateProfile">
    <div>
      <h1 class="header">Welcome to your DineQuest account!</h1>
      <label for="email">Email</label>
      <input id="email" type="text" :value="session.user.email" disabled />
    </div>
    <div>
      <label for="points">Points: </label>
      <span id="points" class="text-3xl font-bold">{{ points }}</span>
    </div>

    <div>
      <button class="button block" @click="roller" :disabled="loading">ROLL FOR A COUPON</button>
    </div>

    <img src="/healthyFood.png"
      width="500" 
     height="350">

    <div>
      <button class="button block" @click="signOut" :disabled="loading">Sign Out</button>
    </div>

  </form>
</template>

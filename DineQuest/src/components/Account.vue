<script setup>
import "../style.css"
import { supabase } from '../supabase'
import { onMounted, ref, toRefs } from 'vue'
import { watch } from 'vue';

const props = defineProps(['session'])
const { session } = toRefs(props)

const loading = ref(true)
const points = ref(0)
const pointsChange = ref('');
const pointsChanged = ref('');
const coupons = ref(0)
const foodItems = ref([])
const orders = ref([])
const showFoodItemsTable = ref(false)
const couponList = ref([])

onMounted(() => {
  getProfile()
  fetchFoodItems()
})

// Retrieves user profile from database
async function getProfile() {
  try {
    loading.value = true
    const { user } = session.value

    const { data, error, status } = await supabase
      .from('profiles')
      .select(`points`)
      .eq('id', user.id) // Filter by user ID
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

//update the profile
async function updateProfile() {
  //tries to get information from supabase
  try {
    loading.value = true
    const { user } = session.value

    //update variables
    const updates = {
      id: user.id,
      email: user.email,
      points: points.value,
      coupons: coupons.value
    }

    //insert updates
    const { error } = await supabase.from('profiles').upsert(updates)

    if (error) throw error
  } catch (error) {
    alert(error.message)
  } finally {
    loading.value = false
  }
}

//sign out
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

// Adds an item to the user's order and updates the profile in the database
async function addItem(item){
  try {
    loading.value = true
    const { user } = session.value

    orders.value.push(item)

    // Define updates object with user data
    const updates = {
      id: user.id,
      email: user.email,
      points: points.value,
      coupons: coupons.value
    }

    const { error } = await supabase.from('profiles').upsert(updates)

    if (error) throw error
  } catch (error) {
    alert(error.message)
  } finally {
    loading.value = false
  }
}

// Pushes all items in the order to the database and updates the user's profile
async function pushAllItems() {
  try {
    loading.value = true;
    const { user } = session.value;

    points.value += orders.value.length * 50;
    orders.value = [];

    const updates = {
      id: user.id,
      email: user.email,
      points: points.value,
      coupons: coupons.value
    };

    const { error } = await supabase.from('profiles').upsert(updates);

    if (error) throw error;
  } catch (error) {
    alert(error.message);
  } finally {
    loading.value = false;
  }
}

// Removes an item from the user's order and updates the profile
async function removeItem(item) {
  try {
    loading.value = true;
    const { user } = session.value;

    const index = orders.value.indexOf(item);

    if (index !== -1) {
      orders.value.splice(index, 1);
    }

    const updates = {
      id: user.id,
      email: user.email,
      points: points.value,
      coupons: coupons.value
    };

    const { error } = await supabase.from('profiles').upsert(updates);

    if (error) throw error;
  } catch (error) {
    alert(error.message);
  } finally {
    loading.value = false;
  }
}


// Fetches food items from the database
async function fetchFoodItems() {
  try {
    loading.value = true
    const { data, error } = await supabase.from('food').select('*')

    if (error) throw error


    if (data) {
      foodItems.value = data // Set the fetched food items
    }
    console.log(data)

  } catch (error) {
    alert(error.message)
  } finally {
    loading.value = false
  }
}
// Toggles the visibility of the food items table
function toggleTableVisibility() {
  showFoodItemsTable.value = !showFoodItemsTable.value
}

//roll for a chance to get a coupon
async function roller() {
  try {
    loading.value = true
    const { user } = session.value

    if (points.value >= 300) {
      points.value -= 300

      // Generate a random number between 1 and 10
      const random_number = Math.floor(Math.random() * 6) + 1;

      // Check if the random number is 10 (10% chance)
      if (random_number === 6) {
        // Display "you win" message
        window.alert("You win a coupon!");
        coupons.value += 1
      }
      else {
        // Display "you win" message
        window.alert("You didn't win a coupon.");
      }
    }

    const updates = {
      id: user.id,
      email: user.email,
      points: points.value,
      coupons: coupons.value
    }

    const { error } = await supabase.from('profiles').upsert(updates)

    if (error) throw error
  } catch (error) {
    alert(error.message)
  } finally {
    loading.value = false
  }
}

// Get coupons
async function getCoupons() {
  try {
    loading.value = true
    const { data, error } = await supabase.from('coupons').select('*')

    if (error) throw error


    if (data) {
      couponList.value = data //get the list of coupons
    }
    //alert(data)

  } catch (error) {
    alert(error.message)
  } finally {
    loading.value = false
  }
}

// Upload the coupon into the database
async function pushCoupon(code) {
  try {
    loading.value = true
    const { error } = await supabase.from('coupons').insert({code: code})

    alert("Save your coupon code!")
    alert(code)

    if (error) throw error;
  } catch (error) {
    alert(error.message);
  } finally {
    loading.value = false;
  }
}

// Spend a counpon
async function spend() {
  try {
    loading.value = true
    const { user } = session.value

    getCoupons()

    if (coupons.value >= 1) {
      coupons.value -= 1

      // Generate a random number
      const ranNum = Math.floor(Math.random() * (999999999 - 100000000 + 1)) + 100000000;
      const index = couponList.value.indexOf(ranNum)

      while(index > -1) {
        ranNum = Math.floor(Math.random() * (999999999 - 100000000 + 1)) + 100000000;
      }

      pushCoupon(ranNum)

    }
    const updates = {
      id: user.id,
      email: user.email,
      points: points.value,
      coupons: coupons.value
    }

    const { error } = await supabase.from('profiles').upsert(updates)

    if (error) throw error
  } catch (error) {
    alert(error.message)
  } finally {
    loading.value = false
  }
}

async function getPoints() {
  try {
    loading.value = true
    const { user } = session.value

    // Input from the user for the reciept
    let input = document.getElementById("reciept").value
    
    // If a valid length add 300 points
    if(input.length > 9) {
      alert("Got 300 Points!")
      points.value += 300
    }
    else {
      alert("Error!")
    }

    // Update
    const updates = {
      id: user.id,
      email: user.email,
      points: points.value,
      coupons: coupons.value
    }

    const { error } = await supabase.from('profiles').upsert(updates)

    if (error) throw error
  } catch (error) {
    alert(error.message)
  } finally {
    loading.value = false
  }
}

// Watches for changes in points value and updates accordingly.
const updatePoints = (newValue, oldValue) => {
  const difference = newValue - oldValue;

  if (difference > 0) {
    pointsChange.value = '+' + difference;
    pointsChanged.value = 'increase';
  } else if (difference === -300) {
    pointsChange.value = '-300';
    pointsChanged.value = 'decrease';
  } else if (difference === -50) {
    pointsChange.value = '-50';
    pointsChanged.value = 'decrease';
  }

  setTimeout(() => {
    pointsChange.value = '';
    pointsChanged.value = '';
  }, 1000);
};

// Watch for changes in points value
watch(points, updatePoints);

document.addEventListener('DOMContentLoaded', fetchFoodItems);
</script>

<template>
  <form id="app" class="container" @submit.prevent="updateProfile" style="background-color: #fffec3; border: 2px solid #000; padding: 20px; border-radius: 10px;">
    <div>
      <h1 class="header">Welcome to your DineQuest account!</h1>
      <label for="email">Email</label>
      <input id="email" type="text" :value="session.user.email" disabled />
    </div>
    <div>
      <label for="receipt">Reciept</label>
      <input id="reciept" type="text"/>
      <button class="button" @click="getPoints" :disabled="loading" style="border: 2px solid #000; padding: 5px; 
      border-radius: 10px; margin-left: 10px; margin-top: 10px">GET POINTS</button>
    </div>
    <div>
      <label for="points">Points: </label>
      <div>
        <span class="points-value">{{ points }}</span>
        <span class="points-change" :class="{ 'flash-green': pointsChanged === 'increase', 'flash-red': pointsChanged === 'decrease' }">{{ pointsChange }}</span>
      </div>    
    </div>
    <div>
      <label for="coupons">Coupons: </label>
      <span id="coupons" class="text-3xl font-bold">{{ coupons }}</span>
    </div>


      <button class="button block" @click="toggleTableVisibility">Menu</button>
      <div>
      <table  v-if="showFoodItemsTable" style="width: 10%; text-align: center; margin-left: auto; margin-right: auto; background-color: #D6EEEE;" >
        <thead>
    <tr>
      <th colspan="5">Click to Add to Order</th>
    </tr>
  </thead>
  <tbody>
    <template v-for="(item, index) in foodItems">
      <tr v-if="index % 5 === 0"></tr>
      <td>
        <button class="button" @click=addItem(item.name)>{{ item.name }}</button>
      </td>
    </template>
  </tbody>
    </table>
    </div>
    <div v-if="!showFoodItemsTable">
      <table style="width: 10%; position: absolute; top: 250px; left: 500px">
      <thead style="text-align: center;">
        <tr>
          <th>
            <div>Orders</div>
            <div>(Click to Remove)</div>
          </th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in orders" :key="item">
          <td>
            <button class="button" @click="removeItem(item)">{{ item }}</button>
          </td>
        </tr>
      </tbody>
      <button class="button" @click="pushAllItems" :disabled="loading" style="border: 2px solid #000; padding: 10px; border-radius: 10px; margin-top: 10px;">Push All Items</button>
    </table>
      <div class="button-container">
        <button class="button" @click="roller" :disabled="loading" style="border: 2px solid #000; padding: 20px; border-radius: 10px;">ROLL FOR A COUPON</button>
        <button class="button" @click="spend" :disabled="loading" style="border: 2px solid #000; padding: 20px; border-radius: 10px;">SPEND A COUPON</button>
      </div>

      <img src="/healthyFood.png"
        width="500" 
      height="350">
      

      <div>
        <button class="button block" @click="signOut" :disabled="loading" style="border: 2px solid #000; padding: 20px; border-radius: 10px;">Sign Out</button>
      </div>
    </div>

  </form>
</template>

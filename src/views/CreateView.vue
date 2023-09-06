<script setup>
import { onMounted, ref } from "vue";
import axios from "axios";
import { useRouter } from "vue-router";
let name = ref(""),
  price = ref(0),
  routerName = ref(""),
  _id = ref("");
const router = useRouter();
onMounted(() => {
  routerName.value = router.currentRoute.value.name;
  _id.value = router.currentRoute.value.params?.id;
  console.log("routerName", routerName.value);
  console.log("_id", _id.value);
  if (routerName.value == "edit") {
    initEdit();
  }
});
const API_PATH = import.meta.env.VITE_API_PATH;
const initEdit = async () => {
  const {data} = await axios.get(`${API_PATH}/getproduct/${_id.value}`);
  name.value = data.data.name;
  price.value = data.data.price;
};

const submit = async () => {
  const req = {
    name: name.value,
    price: price.value
  };
  if(routerName.value == "create"){
  await axios.post(
    `${API_PATH}/create`, 
    req
    ).then((response) => {
    name.value = '';
    price.value = '';
    alert(response.status);
  });
  }else{
    await axios.post(`${API_PATH}/update/${_id.value}`,
req
    ).then((response) => {
      router.push({name : 'home', replace: true});
    });
  }
};
</script>

<template>
  <div class="collapse bg-base-200">
    <input type="radio" name="my-accordion-1" checked="checked" />
    <center>
      <td>
        <font size="5">NAME :</font>
      </td>
      <input
        type="text"
        placeholder="Create Name"
        class="input input-bordered input-info w-full max-w-xs"
        v-model="name"
      /><br /><br />
      <td>
        <font size="5">PRICE :</font>
      </td>
      <input
        type="number"
        placeholder="Create Price"
        class="input input-bordered input-info w-full max-w-xs"
        v-model="price"
      /><br /><br /><br>
    </center>
    <button class="btn btn-info" v-on:click="submit()">Submit</button>
  </div>
</template>

<style scoper></style>

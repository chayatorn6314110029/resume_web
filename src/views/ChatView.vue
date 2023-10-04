<script setup>
import { onMounted, ref, watch, onUpdated } from 'vue';
import {
    app,
    auth,
    database,
    ref as refDb,
    set,
    push,
    onValue,
    remove,
    child,
    get
} from '../firebaseConfig'
let chat = ref("");
let histories = ref([]);
let historyKey = ref(""); // collect select group
const bottomEl = ref(null);
const studentId = "6314110030";

let refChat;
const db = refDb(database, 'all_chat/');
const onSend = () => {
    if (chat.value != '' && historyKey.value != '') {
        push(refDb(database, `all_chat/${historyKey.value}`), {
            "user": studentId,
            "message": chat.value,
            "dataTime": new Date().toISOString()
        });
        chat.value = "";
    }
}

onValue(db, (snapshot) => {
    const data = snapshot.val() ?? [];
    histories.value = data;
})

onUpdated(histories, (newHistories, oldHistories) => {
    bottomEl.value.scrollIntoView({ behavior: 'smooth' });
})

const selectGroup = (key) => {
    historyKey.value = key;
}

let groupChatName = ref("");
const createGroup = () => {
    if (groupChatName.value != '') {
        get(child(db, `${groupChatName.value}` )).then((snapshot) => {
            if(snapshot.exists()){
alert("Cannot create chat because chat is exists.")
            }else {
                push(refDb(database, `all_chat/${groupChatName.value}`), {
            "user": studentId,
            "message": '',
            "dateTime": new Date().toISOString()
        });
            }
        }).catch((err) => {
            console.error(err);
        })
        
        groupChatName.value = '';
    }
}

const deleteGroup = (data) => {
    remove(refDb(database, `all_chat/${data}`));
    window.location.reload();
}
    

</script>

<template>
    <div class="flex">
        <div class="bg-white h-[90vh] w-[30%]">
            <div class="w-[95%] h-[10%]  pt-4">
                <button class="btn btn-accent w-[95%] h-[10%]"  onclick="my_modal_1.showModal()">Add Group</button>
                <dialog id="my_modal_1" class="modal">
                    <div class="modal-box">
                        <div class="from-control w-full">
                            <label class="label">
                                <span class="label-text">Group Name :</span>
                                <input v-model="groupChatName" type="text" placeholder="Enter Group Name">
                            </label>
                        </div>
                        <div class="modal-action">
                            <form method="dialog">
                                <!-- if there is a button in form, it will close the modal -->
                                <button class="btn btn-outline btn-warning">Close</button>
                                <button @click="createGroup()" class="btn btn-outline btn-success">Add</button>
                            </form>
                        </div>
                    </div>
                </dialog>
            </div>
            
                <div class=" overflow-y-scroll w-[95%] h-[90%] card">
                    <div class="card card-side bg-base-100 shadow-xl w-full mb-4" style="cursor: pointer;"
                        v-for="(group, index) in histories" :key="index" data-theme="dark" @click="selectGroup(index)">
                        <div class="card-body">
                            <h2 class="card-title">{{ index }}</h2><br>
                            <p>{{ group[Object.keys(group)[Object.keys(group).length - 1]]?.message }}</p>                           
                        </div>

                        <button class="btn w-[30%] h-full" @click="deleteGroup(index)">Delete Group</button>
                        
                    </div>
                </div>

            
        </div>

        <div class="bg-gray-50 h-[90vh] w-full">
            <div class="overflow-y-scroll h-[90%] p-5 card" data-theme="dark">
                <div v-for="(history, index) in histories[historyKey]"
                    :class="`chat ${history.user == studentId ? 'chat-end' : 'chat-start'}`" :key="index">
                    <div class="chat-header">
                        {{ history.user }}
                        <time class="text-xs opacity-50"> {{ history.dateTime }}</time>
                    </div>
                    <div className="chat-bubble">{{ history.message }}</div>
                </div>
                <div ref="bottomEl"></div>
            </div>
            <br>
            <div class="flex h-[10%] gap-4">
                <input v-on:keyup.enter="onSend" v-model="chat" type="text" placeholder="Enter Chat"
                    class="input input-bordered w-[80%] h-full">
                <button @click="onSend" class=" btn btn-accent w-[20%] h-full">Send</button>
            </div>
        </div>
    </div>
</template>
<style scoped></style>
<template>
  <!-- <div class="p-6">
    <div class="flex flex-col-reverse overflow-auto h-96 mb-4">
      <div v-for="(chat, index) in chats" :key="index" class="my-2 flex">
        <div :class="chat.sender === 'user' ? 'ml-auto bg-blue-500 text-white' : 'mr-auto bg-gray-300 max-w-80'"
          class="p-2 rounded">
          {{ chat.message }}
        </div>
      </div>
    </div>
    <div class="p-4 fixed inset-x-0 bottom-10">
      <div>
        <textarea v-model="message" placeholder="Type your message here..." class="border p-2 w-full"
          :disabled="loading"></textarea>
      </div>
      <button @click="sendChat" class="bg-blue-500 text-white p-2 mt-2" :disabled="loading">
        <div v-if="loading" class="animate-spin mr-2 inline-block w-4 h-4 border-2 rounded-full" role="status">
          <span class="visually-hidden"></span>
        </div>
        Send
      </button>
    </div>

  </div> -->
  <!-- Component Start -->
	<div class="flex flex-col flex-grow w-full max-w-xl bg-white shadow-xl rounded-lg overflow-hidden">
		<div class="flex flex-col flex-grow h-0 p-4 overflow-auto">
			<template v-for="(chat, index) in chats" :key="index">
				<!-- Message from system -->
				<div class="flex w-full mt-2 space-x-3 max-w-xs" v-if="chat.sender === 'system'">
					<div class="flex-shrink-0 h-10 w-10 rounded-full bg-gray-300"></div>
					<div>
						<div class="bg-gray-300 p-3 rounded-r-lg rounded-bl-lg">
							<p class="text-sm">{{ chat.message }}</p>
						</div>
						<span class="text-xs text-gray-500 leading-none">2 min ago</span>
					</div>
				</div>
				<!-- Message from user -->
				<div class="flex w-full mt-2 space-x-3 max-w-xs ml-auto justify-end" v-if="chat.sender === 'user'">
					<div>
						<div class="bg-blue-600 text-white p-3 rounded-l-lg rounded-br-lg">
							<p class="text-sm">{{ chat.message }}</p>
						</div>
						<span class="text-xs text-gray-500 leading-none">2 min ago</span>
					</div>
					<div class="flex-shrink-0 h-10 w-10 rounded-full bg-gray-300"></div>
				</div>
			</template>
		</div>
		
		<div class="bg-gray-300 p-4">
			<textarea class="flex items-center h-10 w-full rounded px-3 text-sm" type="text" placeholder="Type your message…" v-model="message" :disabled="loading" 
    @keydown.enter.prevent="sendChat()"></textarea>
			<button @click="sendChat" class="bg-blue-500 text-white p-2 mt-2" :disabled="loading">
				<div v-if="loading" class="animate-spin mr-2 inline-block w-4 h-4 border-2 rounded-full" role="status">
					<span class="visually-hidden"></span>
				</div>
					Send
			</button>
		</div>
	</div>
	<!-- Component End  -->
</template>


<script>
import { ref } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const message = ref('');
    const chats = ref([]);
    const loading = ref(false);
    const threadId = ref(null);
    const runId = ref(null);
    const polling = ref(null);
    const api_url = 'http://164.68.105.132:8000';

    const sendChat = async () => {
      if (!message.value) return;
      loading.value = true;
      chats.value.push({ sender: 'user', message: message.value });
      const messageToSend = message.value;

      try {
        message.value = '';
        const response = await axios.post(`${api_url}/chat`, {
          thread_id: threadId.value || '',
          message: messageToSend,
        });


        runId.value = response.data.run_id;
        threadId.value = response.data.thread_id

        startPolling();
      } catch (error) {
        console.error(error);
      }
    };

    const startPolling = () => {
      if (polling.value) {
        clearInterval(polling.value);
        loading.value = false;
      }

      polling.value = setInterval(async () => {
        await pollResponse();
      }, 5000); // Poll every 5 seconds
    };

    const pollResponse = async () => {
      try {
        const response = await axios.get(`${api_url}/chat/${threadId.value || ''}/response/${runId.value || ''}`);

        if (response.data.status === 'completed') {
          clearInterval(polling.value);
          polling.value = null;

          // Append system response to chat
          const message = response.data.messages.data[0].content[0].text.value;
          chats.value.push({ sender: 'system', message: message });
          loading.value = false;
        }

      } catch (error) {
        console.error(error);
        clearInterval(polling.value);
        loading.value = false;
      }
    };

    return { message, chats, loading, sendChat, pollResponse };
  },
};


</script>

<style>
/* Add additional styles here */
</style>

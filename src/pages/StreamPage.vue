<template>
	<div class="container">
		<p style="font-size: 18px; font-weight: bold;">接入DeepSeek大模型，本地部署聊天界面</p>
		<div class="input-group">
			<el-input type="text" id="messageInput" placeholder="请输入问题..." v-model.trim="message" ref="myinput"
				style="width: 500px;"></el-input>
			<el-button id="submitBtn" @click="mes">提交</el-button>
		</div>
		<div id="response" ref="responseDiv">
			{{ content }}
		</div>
	</div>
</template>

<script setup lang="js">
import { ref } from 'vue';
const responseDiv = ref(null);
const myinput = ref(null);
const message = ref('');
// AI回答的结果
const content = ref('');
const mes = () => {
	console.log(responseDiv.value);
	const eventSource = new EventSource(
		`http://${import.meta.env.VITE_DEV_URL_SAMESITE}/deepseek/chatStream?msg=${encodeURIComponent(message.value)}`);

	let currentLine = '';
	eventSource.onmessage = function (event) {
		const data = event.data;
		for (let char of data) {
			if (currentLine.length >= 30) {
				content.value = currentLine + '\n';
				currentLine = '';
			}
			currentLine += char;
		}
		// 处理剩余字符
		if (data.length > 0 && currentLine.length > 0) {
			content.value += currentLine;
			currentLine = '';
		}
	};
	eventSource.onerror = function (error) {
		console.error('EventSource failed:', error);
		eventSource.close();
	};
	myinput.value.addEventListener('input', function () {
		eventSource.close();
	});
}
</script>

<style></style>

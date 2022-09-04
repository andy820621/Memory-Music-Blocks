<script setup>
import { onMounted, reactive, ref } from "vue";

const blocksContainer = ref(null);
const inputStatus = ref(null);
const startContainer = ref(null);

const audioDatas = reactive([]); // length 25
for (let i = 1; i <= 15; i += 0.5) {
	if (!getAudioObject(i)) continue;
	audioDatas.push(getAudioObject(i));
	if (audioDatas.length === 25) break;
}

const blockData = ref([
	{ element: null, id: "1", pitch: "0", color: "0 100% 66%" },
	{ element: null, id: "2", pitch: "2", color: "43 100% 58%" },
	{ element: null, id: "3", pitch: "4", color: "218 46% 55%" },
	{ element: null, id: "4", pitch: "6", color: "48 36% 77%" },
]);
const soundSets = reactive({
	correct: [0, 4, 8, 13],
	wrong: [2, 6, 9, 12],
	gameover: [4, 8, 24],
});
const messageBox = reactive({
	correct: `Correct!!
	Let's go to the next level~`,
	wrong: `😛😛😛 You Wrong!! Try it again!! 😛😛😛`,
	gameover: `👎👎👎 You fucked up!! Go back to your mother!! 👎👎👎`,
});
const levelQuestions = reactive([
	"0246",
	"04642",
	"264062",
	// "2123142",
	// "31423421",
	// "341243412",
	// "4132412431",
	// "13412434124",
]);

const allOn = ref(false);
const userLifes = ref(5);
const currentLevel = ref(0);
const mode = ref("waiting");
const userInputAnswer = ref("");

const answer = ref(levelQuestions[currentLevel.value]);
let checkAnswer = answer.value.split("");
let questionTimer;

const Message = ref("");

function blockHandler(pitch) {
	if (mode.value === "userInputting") {
		blockPlaySound(pitch);
		checkCrrect(pitch);
	}
}

function blockPlaySound(pitch) {
	let block = blockData.value.find((block) => block.pitch === pitch)?.element;
	let audio = audioDatas[pitch];
	audio.currentTime = 0;
	audio.play();

	block.classList.add("flicker");
	setTimeout(() => {
		if (allOn.value) return;
		block.classList.remove("flicker");
	}, 100);
}

function getAudioObject(pitch) {
	if (pitch === 7.5 || pitch === 10.5) return;
	return new Audio(`https://awiclass.monoame.com/pianosound/set/${pitch}.wav`);
}
function getElement() {
	blockData.value.forEach(
		(block) => (block.element = document.querySelector(`#block${block.id}`))
	);
}

function turnAllOn() {
	allOn.value = true;
	document
		.querySelectorAll(".block")
		.forEach((block) => block.classList.add("flicker"));
}
function turnAllOff() {
	allOn.value = false;
	document
		.querySelectorAll(".block")
		.forEach((block) => block.classList.remove("flicker"));
}
function playSound(type) {
	let sets = soundSets[type];
	sets.forEach((index) => {
		audioDatas[index].currentTime = 0;
		audioDatas[index].play();
	});
}
// startGame
function showQuestion() {
	mode.value = "showingQuestion";
	// reset question
	answer.value = levelQuestions[currentLevel.value];
	checkAnswer = answer.value.split("");

	let notes = answer.value.split("");
	questionTimer = setInterval(() => {
		let pitch = notes.shift();
		blockPlaySound(pitch);
		if (!notes.length) {
			setModeToUserInputting();
			clearInterval(questionTimer);
		}
	}, 400);
}

function setModeToUserInputting() {
	mode.value = "userInputting";
	userInputAnswer.value = "";
}

function checkCrrect(pitch, block) {
	// if wrong
	if (pitch !== checkAnswer[0]) {
		userLifes.value--;
		// if gameover
		if (userLifes.value === 0) {
			showMessage("gameover");
			startContainer.value.classList.remove("hidden");
		} else {
			showMessage("wrong");
			setTimeout(() => {
				clean();
				showQuestion();
			}, 2000);
		}
		return;
	}
	// if correct
	userInputAnswer.value += checkAnswer.shift();
	// if all correct
	if (checkAnswer.length === 0) {
		showMessage("correct");
		setTimeout(() => {
			currentLevel.value++;
			clean();
			showQuestion();
		}, 2000);
	}
}

function showMessage(correctOrWrong) {
	inputStatus.value.classList.add(correctOrWrong);
	mode.value = "waiting";
	Message.value = messageBox[correctOrWrong];
	setTimeout(() => {
		turnAllOn();
		playSound(correctOrWrong);
	}, 500);
}

function clean() {
	Message.value = "";
	turnAllOff();
	inputStatus.value.className = "inputStatus";
	userInputAnswer.value = "";
}

function startButtonHandler(e) {
	startContainer.value.classList.add("hidden");
	userLifes.value = 5;
	currentLevel.value = 0;
	clean();
	showQuestion();
}

onMounted(() => {
	getElement();
});
</script>

<template>
	<div class="infos">
		<h1>Memory Blocks</h1>
		<h3 class="status">Level {{ currentLevel + 1 }}</h3>
		<h3 class="message" v-if="Message">{{ Message }}</h3>
	</div>

	<main>
		<ul class="blocks" ref="blocksContainer">
			<li
				v-for="block in blockData"
				class="block"
				:style="{
					'--block-color': block.color,
					cursor: mode === 'userInputting' ? 'pointer' : 'not-allowed',
				}"
				:id="'block' + block.id"
				:key="block.id"
				@click="blockHandler(block.pitch)"
			></li>
		</ul>

		<div class="bottom">
			<ul class="inputStatus" ref="inputStatus">
				<li
					class="circle"
					v-for="(circle, index) in answer"
					:class="{ correct: index < userInputAnswer.length }"
					:key="circle + index"
				></li>
			</ul>

			<ul class="lifes">
				<li
					class="life"
					v-for="life in Array.from({ length: userLifes }, (v, i) => i)"
				>
					🧡
				</li>
			</ul>
		</div>
	</main>
	<div class="start" ref="startContainer">
		<button @click="startButtonHandler">Start</button>
	</div>
</template>

<style lang="scss">
$colorRed: hsl(0, 100%, 66%);
$colorYellow: hsl(43, 100%, 58%);
$colorBlue: hsl(218, 46%, 55%);
$colorWhite: hsl(48, 36%, 77%);

ul {
	list-style: none;
}

.infos {
	position: fixed;
	top: 2rem;
	left: 2.4rem;
	h1 {
		font-size: 2rem;
		letter-spacing: 1px;
		margin: 0 auto;
		line-height: 1.2;
	}
	h3 {
		font-weight: 500;
		color: $colorRed;
	}
}
main {
	width: min-content;
}
.blocks {
	--grid-number: 2;
	--block-size: 8rem;
	--grid-gap: 2rem;

	display: grid;
	grid-template-columns: repeat(var(--grid-number), var(--block-size));
	grid-template-rows: repeat(var(--grid-number), var(--block-size));
	gap: var(--grid-gap);
	margin: 0;
	padding: 0;

	.block {
		--block-color: 0 100% 66%;

		width: var(--block-size);
		height: var(--block-size);
		left: calc(
			var(--grid-gap) + var(--x) * (var(--grid-gap) + var(--block-size))
		);
		top: calc(
			var(--grid-gap) + var(--y) * (var(--grid-gap) + var(--block-size))
		);

		border: 1px solid;
		border-color: hsl(var(--block-color));
		filter: brightness(80%);

		box-shadow: 0 0 24px 4px hsl(var(--block-color) / 0.14);
		animation: shadow-animation 1.2s infinite alternate-reverse;

		position: relative;
		&::after {
			content: "";
			position: absolute;
			inset: 0;
			transition: 0.5s 0.1s;
		}
		&.flicker::after {
			transition: 0s;
			background-color: hsl(var(--block-color));
			box-shadow: 0 0 24px 4px hsl(var(--block-color) / 0.44);
		}
	}
}

@keyframes shadow-animation {
	100% {
		box-shadow: 0 0 24px 4px hsl(var(--block-color) / 0.44);
	}
}

// Bottom Design
.bottom {
	width: 100%;
	display: flex;
	justify-content: space-between;
	align-items: center;
	& > ul {
		padding: 0;
	}
}
// InputStatus Design
.inputStatus {
	margin-right: auto;
	display: flex;
	column-gap: 8px;
	.circle {
		background-color: #fff;
		width: 8px;
		height: 8px;
		opacity: 0.3;
		border-radius: 50%;
		&.correct {
			opacity: 1;
		}
	}
	&.correct .circle {
		background-color: $colorBlue;
	}
	&.wrong .circle {
		background-color: $colorRed;
	}
	&.gameover .circle {
		background-color: transparent;
	}
}
// lifes
.lifes {
	display: flex;
	margin-left: auto;
	gap: 0.24rem;
}

.start {
	position: fixed;
	inset: 0;
	backdrop-filter: blur(0.8px);

	display: grid;
	place-items: center;

	&.hidden {
		opacity: 0;
		visibility: hidden;

		button {
			opacity: 0;
			visibility: hidden;
		}
	}

	button {
		padding: 0.8rem 2.4rem;
		border: 2px solid #eee;
		color: #eee;
		font-size: 2.4vw;
		background-color: transparent;
		border-radius: 2.5rem;
		transition: background-color 0.3s;
		&:active {
			transform: translateY(2.4px);
		}
		&:hover {
			background-color: rgba(240, 240, 240, 0.08);
		}
	}
}
</style>

<script setup>
import { nextTick, reactive, ref, computed } from "vue";

const blocksContainer = ref(null);
const blockElements = ref(null);
const inputStatus = ref(null);
const startContainer = ref(null);

const audioDatas = reactive([]); // length 25
for (let i = 1; i <= 15; i += 0.5) {
	if (!getAudioObject(i)) continue;
	audioDatas.push(getAudioObject(i));
	if (audioDatas.length === 25) break;
}

const allOn = ref(false);
const userLifes = ref(5);
const currentLevel = ref(0);
const mode = ref("waiting");
const userInputAnswer = ref([]);
const blockDataObjects = reactive([
	{
		questionNumber: 3,
		size: {
			gridNumber: 2,
			blockSize: "min(8rem, calc(45vw - 1rem))",
			gridGap: "min(2rem, 9.5vw)",
		},
		data: [
			{ id: "1", pitch: "0", color: "0 100% 66%" },
			{ id: "2", pitch: "2", color: "43 100% 58%" },
			{ id: "3", pitch: "4", color: "218 46% 55%" },
			{ id: "4", pitch: "6", color: "48 36% 77%" },
		],
	},
	{
		questionNumber: 3,
		size: {
			gridNumber: 3,
			blockSize: "min(8rem, calc(30vw - 1rem))",
			gridGap: "min(1.5rem, 5vw)",
		},
		data: [
			{ id: "1", pitch: "0", color: "0 100% 66%" },
			{ id: "2", pitch: "1", color: "43 100% 58%" },
			{ id: "3", pitch: "2", color: "218 46% 55%" },
			{ id: "4", pitch: "3", color: "48 36% 77%" },
			{ id: "5", pitch: "4", color: "326 40% 60%" },
			{ id: "6", pitch: "5", color: "99 44% 59%" },
			{ id: "7", pitch: "6", color: "260 55% 70%" },
			{ id: "8", pitch: "7", color: "60 55% 50%" },
			{ id: "9", pitch: "8", color: "30 45% 60%" },
		],
	},
	{
		questionNumber: 4,
		size: {
			gridNumber: 4,
			blockSize: "min(8rem, calc(22.5vw - 0.9rem))",
			gridGap: "min(1.2rem, 3vw)",
		},
		data: [
			{ id: "1", pitch: "0", color: "0 100% 66%" },
			{ id: "2", pitch: "1", color: "43 100% 58%" },
			{ id: "3", pitch: "2", color: "218 46% 55%" },
			{ id: "4", pitch: "3", color: "48 36% 77%" },
			{ id: "5", pitch: "4", color: "326 40% 60%" },
			{ id: "6", pitch: "5", color: "99 44% 59%" },
			{ id: "7", pitch: "6", color: "260 55% 70%" },
			{ id: "8", pitch: "7", color: "60 55% 50%" },
			{ id: "9", pitch: "8", color: "30 45% 60%" },
			{ id: "10", pitch: "9", color: "226 76% 60%" },
			{ id: "11", pitch: "10", color: "97 76% 62%" },
			{ id: "12", pitch: "11", color: "342 76% 62%" },
			{ id: "13", pitch: "12", color: "253 81% 65%" },
			{ id: "14", pitch: "13", color: "67 76% 62%" },
			{ id: "15", pitch: "14", color: "300 56% 70%" },
			{ id: "16", pitch: "15", color: "174 76% 62%" },
		],
	},
	{
		questionNumber: 5,
		size: {
			gridNumber: 5,
			blockSize: "min(6rem, calc(18vw - 0.8rem))",
			gridGap: "min(1rem, 2.6vw)",
		},
		data: [
			{ id: "1", pitch: "0", color: "0 100% 66%" },
			{ id: "2", pitch: "1", color: "43 100% 58%" },
			{ id: "3", pitch: "2", color: "218 46% 55%" },
			{ id: "4", pitch: "3", color: "48 36% 77%" },
			{ id: "5", pitch: "4", color: "326 40% 60%" },
			{ id: "6", pitch: "5", color: "99 44% 59%" },
			{ id: "7", pitch: "6", color: "260 55% 70%" },
			{ id: "8", pitch: "7", color: "60 55% 50%" },
			{ id: "9", pitch: "8", color: "30 45% 60%" },
			{ id: "10", pitch: "9", color: "226 76% 60%" },
			{ id: "11", pitch: "10", color: "97 76% 81%" },
			{ id: "12", pitch: "11", color: "342 76% 62%" },
			{ id: "13", pitch: "12", color: "253 81% 65%" },
			{ id: "14", pitch: "13", color: "67 76% 62%" },
			{ id: "15", pitch: "14", color: "300 56% 70%" },
			{ id: "16", pitch: "15", color: "174 76% 62%" },
			{ id: "17", pitch: "16", color: "222 51% 67%" },
			{ id: "18", pitch: "17", color: "36 66% 64%" },
			{ id: "19", pitch: "18", color: "0, 70%, 55%" },
			{ id: "20", pitch: "19", color: "118 58% 61%" },
			{ id: "21", pitch: "20", color: "64 81% 50%" },
			{ id: "22", pitch: "21", color: "0 81% 65%" },
			{ id: "23", pitch: "22", color: "190 70% 44%" },
			{ id: "24", pitch: "23", color: "74 61% 71%" },
			{ id: "25", pitch: "24", color: "226 100% 81%" },
		],
	},
]);

const currentDataIndex = ref(0);
const currentBlockDataObject = computed(
	() => blockDataObjects[currentDataIndex.value]
);
const currentBlockData = computed(() => currentBlockDataObject.value.data);
const soundSets = reactive({
	correct: [0, 4, 8, 13],
	wrong: [2, 6, 9, 12],
	gameover: [4, 8, 24],
	win: [
		[0, 4, 8, 13],
		[2, 6, 9, 12],
		[4, 8, 24],
	],
});
const messageBox = reactive({
	correct: `Correct!!
	Let's go to the next level~`,
	wrong: `😛😛😛 You Wrong!! Try it again!! 😛😛😛`,
	gameover: `👎👎👎 You fucked up!! Go back to your mother!! 👎👎👎`,
	win: `🤙🤙🤙 Congratulations!! You Win this Kuso Game!!🤙🤙🤙`,
});
const levelQuestions = reactive([]);
// produce questions
produceQuestions();
function produceQuestions() {
	levelQuestions.length = 0; // ensure questions array is empty.
	blockDataObjects.forEach((blockDataObject) => {
		let questionNumber = blockDataObject.questionNumber;
		let currentPitchSets = getBlockDataPitchSets(blockDataObject.data);

		for (let i = 0; i < questionNumber; i++) {
			let question = getQuestion(currentPitchSets, currentLevel.value + 4);
			levelQuestions.push(question);
			currentLevel.value++;
		}
	});
	// after produced, init data
	currentLevel.value = 0;
}
function getBlockDataPitchSets(currentData) {
	return currentData.map((block) => block.pitch);
}
function getQuestion(pitchSets, questionNumber) {
	let result = [];
	for (let i = 0; i < questionNumber; i++) {
		let index = parseInt(pitchSets.length * Math.random());
		result.push(pitchSets[index]);
	}
	return result;
}

const answer = ref(levelQuestions[currentLevel.value]);
const checkAnswer = ref([]);
checkAnswer.value = [...answer.value];

const Message = ref("");

function blockHandler(pitch) {
	if (mode.value === "userInputting") {
		blockPlaySound(pitch);
		checkCrrect(pitch);
	}
}

function blockPlaySound(pitch) {
	let blockIndex = currentBlockData.value.findIndex(
		(block) => block.pitch === pitch
	);
	let blockElement = blockElements.value[blockIndex];
	let audio = audioDatas[pitch];
	audio.currentTime = 0;
	audio.play();

	blockElement.classList.add("flicker");
	setTimeout(() => {
		if (allOn.value) return;
		blockElement.classList.remove("flicker");
	}, 100);
}

function getAudioObject(pitch) {
	if (pitch === 7.5 || pitch === 10.5) return;
	return new Audio(`https://awiclass.monoame.com/pianosound/set/${pitch}.wav`);
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
	if (type === "win") {
		let time = 0;
		sets.forEach((setArray) => {
			setTimeout(() => {
				setArray.forEach((index) => {
					audioDatas[index].currentTime = 0;
					audioDatas[index].play();
				});
			}, time);
			time += 100;
		});
	} else {
		sets.forEach((index) => {
			audioDatas[index].currentTime = 0;
			audioDatas[index].play();
		});
	}
}
// startGame
async function showQuestion() {
	mode.value = "showingQuestion";

	let time = 0;
	for (let i = 0; i < answer.value.length; i++) {
		time += 400;
		setTimeout(async () => {
			blockPlaySound(answer.value[i]);
			await nextTick();
		}, time);
	}
	await nextTick();
	setTimeout(setModeToUserInputting, time + 200);
}

function setModeToUserInputting() {
	mode.value = "userInputting";
	userInputAnswer.value = [];
}

function checkCrrect(pitch, block) {
	// if wrong
	if (pitch !== checkAnswer.value[0]) {
		userLifes.value--;
		// if gameover
		if (userLifes.value === 0) {
			showMessage("gameover");
			startContainer.value.classList.remove("hidden");
			produceQuestions();
		} else {
			showMessage("wrong");
			setTimeout(async () => {
				clean();
				updateAnswer();
				await nextTick();
				showQuestion();
			}, 2000);
		}
		return;
	}
	// if correct
	userInputAnswer.value.push(checkAnswer.value.shift());
	// if all correct
	if (checkAnswer.value.length === 0) {
		showMessage("correct");
		setTimeout(async () => {
			currentLevel.value++;
			if (currentLevel.value === levelQuestions.length) {
				showMessage("win");
				startContainer.value.classList.remove("hidden");
				return;
			}
			userLifes.value = 5;
			clean();
			currentBlockDataObject.value.questionNumber--;
			if (currentBlockDataObject.value.questionNumber === 0) {
				currentDataIndex.value++;
			}
			await nextTick();
			updateAnswer();
			showQuestion();
		}, 2000);
	}
}
function updateAnswer() {
	answer.value = levelQuestions[currentLevel.value];
	checkAnswer.value = [...answer.value];
}

function showMessage(correctOrWrong) {
	inputStatus.value.classList.add(correctOrWrong);
	setTimeout(() => (mode.value = "waiting"), 100);
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
	userInputAnswer.value = [];
}

async function startButtonHandler(e) {
	if (userLifes.value === 0) {
		userLifes.value = 5;
		currentLevel.value = 0;
		currentDataIndex.value = 0;
		clean();
	}
	startContainer.value.classList.add("hidden");

	updateAnswer();
	await nextTick();
	setTimeout(showQuestion, 300);
}
</script>

<template>
	<div class="infos">
		<h1>
			Memory <br />
			Blocks
		</h1>
		<h3 class="status">Level {{ currentLevel + 1 }}</h3>
		<h3 class="message" v-if="Message">{{ Message }}</h3>
	</div>

	<main>
		<ul
			class="blocks"
			ref="blocksContainer"
			:style="{
				'--grid-number': currentBlockDataObject.size.gridNumber,
				'--block-size': currentBlockDataObject.size.blockSize,
				'--grid-gap': currentBlockDataObject.size.gridGap,
			}"
		>
			<li
				v-for="(block, index) in currentBlockData"
				class="block"
				:class="{ userInputting: mode === 'userInputting' }"
				:style="{
					'--block-color': block.color,
					animationDelay: -index * 0.5 + 's',
				}"
				:id="'block' + block.id"
				:key="block.id"
				@click="blockHandler(block.pitch)"
				ref="blockElements"
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
		<button @click="startButtonHandler">
			{{ userLifes === 0 ? "ReStart" : "Start" }}
		</button>
	</div>
</template>

<style lang="scss">
$colorRed: hsl(0, 100%, 66%);
$colorYellow: hsl(43, 100%, 58%);
$colorBlue: hsl(218, 46%, 55%);
$colorWhite: hsl(48, 36%, 77%);
$colorTry: hsl(226, 100%, 61%);

ul {
	list-style: none;
}

.infos {
	position: fixed;
	top: 2rem;
	left: 2.4rem;
	z-index: 9999;
	h1 {
		font-size: 2rem;
		letter-spacing: 1px;
		margin: 0 auto;
		line-height: 1.2;
	}
	h3 {
		font-weight: 500;
		&.status {
			color: rgb(77, 182, 172);
		}
		&.message {
			display: block;
			position: fixed;
			left: 50%;
			border-radius: 5px;
			transform: translateX(-50%);
			text-align: center;
			color: #fff;
			padding: 0.24rem 0.8rem;
			font-size: 1.2rem;
			background-color: hsl(0 81% 50% / 0.24);
			backdrop-filter: contrast(81%) blur(0.8rem);
		}
	}
}
main {
	width: min-content;
}
.blocks {
	--grid-number: 5;
	--block-size: min(6rem, calc(18vw - 0.8rem));
	--grid-gap: min(1rem, 2.6vw);

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
		animation: shadow-animation 1.5s infinite alternate-reverse;

		cursor: not-allowed;
		transition: 0.3s;

		position: relative;
		&::after {
			content: "";
			position: absolute;
			inset: 0;
			transition: 0.5s 0.1s;
			box-shadow: inset 0 0 2.4em rgba(240, 240, 240, 0.24);
		}
		&.flicker::after {
			transition: 0s;
			background-color: hsl(var(--block-color));
			box-shadow: 0 0 24px 4px hsl(var(--block-color) / 0.5);
		}
	}
	.userInputting {
		cursor: pointer;
		&:hover {
			background-color: hsl(var(--block-color) / 0.14);
		}
	}
}

@keyframes shadow-animation {
	100% {
		box-shadow: 0 0 24px 4px hsl(var(--block-color) / 0.5);
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
	&.win .circle {
		background-color: $colorBlue;
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

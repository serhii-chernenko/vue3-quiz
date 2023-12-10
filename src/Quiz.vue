<template>
  <div class="flex h-screen items-center justify-center">
    <div v-if="isResult">
      <div class="text-center">
        <h1 class="font-bold">Result</h1>
        <p>
          You scored {{ score }} out of {{ questions.length
          }}{{ scorePercentage }}
        </p>
        <div class="my-2">
          <button
            type="button"
            @click="retry"
            class="py-2 px-5 rounded bg-blue-500 text-white cursor-pointer transition-colors hover:bg-blue-600"
          >
            Retry
          </button>
        </div>
        <div>
          <button
            type="button"
            @click="showWrongAnswers = !showWrongAnswers"
            v-if="score < questions.length"
            class="mb-5 text-blue-500 cursor-pointer transition-colors hover:text-blue-600"
          >
            {{ showWrongAnswers ? "Hide" : "Show" }} wrong answers
          </button>
        </div>
      </div>
      <template v-if="showWrongAnswers && score < questions.length">
        <ul v-for="(question, index) in wrongAnswers" :key="index">
          <li class="my-5">
            <p>
              <strong>{{ question.question }}</strong>
            </p>
            <ul>
              <li class="text-green-700">
                Correct answer: {{ question.correctAnswer }}
              </li>
              <li class="text-red-700">
                Your answer: {{ question.options[question.answer] }}
              </li>
            </ul>
          </li>
        </ul>
      </template>
    </div>
    <form v-else @submit.prevent="submit">
      <fieldset>
        <legend class="mb-5">
          <strong>{{ getCurrentQuestion.question }}</strong>
        </legend>
      </fieldset>
      <fieldset>
        <label
          v-for="(option, index) in getCurrentQuestion.options"
          :key="index"
          class="group flex items-center p-1 cursor-pointer"
        >
          <input
            type="radio"
            name="answer"
            @change="currentAnswerIndex = index"
            :checked="currentAnswerIndex === index"
            class="appearance-none"
          />
          <ins
            class="relative block w-[16px] h-[16px] rounded-full border border-solid border-blue-500"
          >
            <span
              class="absolute top-1/2 left-1/2 block w-[10px] h-[10px] rounded-full bg-blue-500 transform -translate-x-1/2 -translate-y-1/2 scale-0"
              :class="{
                'scale-100': currentAnswerIndex === index,
                'transition-transform group-hover:scale-100':
                  currentAnswerIndex !== index,
              }"
            ></span>
          </ins>
          <span class="ml-2">{{ option }}</span>
        </label>
      </fieldset>
      <fieldset class="mt-5">
        <button
          type="button"
          class="mr-5 text-gray-600 cursor-pointer transition-colors hover:text-black"
          v-if="currentQuestionIndex > 0"
          @click="prevQuestion"
        >
          Previous
        </button>
        <button
          type="submit"
          class="py-2 px-5 rounded bg-blue-500 text-white cursor-pointer transition-colors hover:bg-blue-600"
        >
          {{ buttonLabel }}
        </button>
      </fieldset>
    </form>
  </div>
</template>
<script setup>
import { computed, shallowRef, reactive } from "vue";

const questions = reactive([
  {
    question: "What is the capital of France?",
    options: ["London", "Berlin", "Paris", "Rome"],
    correctAnswer: "Paris",
  },
  {
    question: "Which planet is closest to the sun?",
    options: ["Earth", "Mars", "Venus", "Mercury"],
    correctAnswer: "Mercury",
  },
]);

const isResult = shallowRef(false);
const showWrongAnswers = shallowRef(false);
const score = shallowRef(0);
const scorePercentage = computed(() => {
  const result = Math.round((score.value / questions.length) * 100);

  return result ? ` (${result}%)` : "";
});
const currentQuestionIndex = shallowRef(0);
const getCurrentQuestion = computed(() => {
  return questions[currentQuestionIndex.value];
});
const currentAnswerIndex = shallowRef(0);
const buttonLabel = computed(() => {
  return currentQuestionIndex.value === questions.length - 1
    ? "Submit"
    : "Next";
});
const wrongAnswers = computed(() => {
  return questions.filter(
    (question) => question.options[question.answer] !== question.correctAnswer,
  );
});

const prevQuestion = () => {
  currentQuestionIndex.value--;
  currentAnswerIndex.value = questions[currentQuestionIndex.value].answer;
};

const submit = () => {
  questions[currentQuestionIndex.value].answer = currentAnswerIndex.value;

  if (
    questions[currentQuestionIndex.value].options[currentAnswerIndex.value] ===
    questions[currentQuestionIndex.value].correctAnswer
  ) {
    score.value++;
  }

  if (currentQuestionIndex.value === questions.length - 1) {
    isResult.value = true;

    return false;
  }

  currentQuestionIndex.value++;
  currentAnswerIndex.value = 0;
};

const retry = () => {
  isResult.value = false;
  score.value = 0;
  currentQuestionIndex.value = 0;
  currentAnswerIndex.value = 0;
  questions.forEach((question) => {
    delete question.answer;
  });
};
</script>

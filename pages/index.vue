<template>
  <div class="container">
    <div class="row">
      <div class="col mt-5">
        <h1 class="title">suggest</h1>
        <p>userEnteredText: {{ userEnteredText }}</p>

        <!-- https://github.com/KazanExpress/vue-simple-suggest#component-api -->
        <!-- https://github.com/KazanExpress/vue-simple-suggest/blob/master/example/src/App.vue -->
        <click-watcher @click-outside="$refs.wikipediaTypeAhead.hideList(); showLoading = false;">
          <vue-simple-suggest
            ref="wikipediaTypeAhead"
            v-model="userEnteredText"
            display-attribute="text"
            :debounced="500"
            :list="getSuggestionList"
            :styles="autoCompleteStyle"
            @select="handleSelectedSuggestion"
            @request-start="showLoading = true"
            @show-list="showLoading = false;"
          >
          </vue-simple-suggest>
        </click-watcher>
        <ul v-show="showLoading === true" class="position-absolute list-group">
          <li class="list-group-item" style="font-size: 48px;">
            <em class="fas fa-spinner fa-spin mr-2"></em>
            searching...
          </li>
        </ul>
      </div>
    </div>
    <div class="row">
      <div class="col mt-5">
        <div v-show="selection.text && selection.text === userEnteredText">
          <p>text: {{selection.text}}</p>
          <p>description: {{selection.description}}</p>
          <p>link: {{selection.link}}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import VueSimpleSuggest from 'vue-simple-suggest';

async function getSuggestionList(query) {
  this.$refs.wikipediaTypeAhead.clearSuggestions();
  console.time('getSuggestionList ajax');
  const limit = 25;
  let response;
  try {
    response = await this.$axios({
      method: 'GET',
      url: `https://en.wikipedia.org/w/api.php?origin=*&action=opensearch&namespace=*&search=${query}&limit=${limit}&namespace=0&format=json`,
    });
    console.timeEnd('getSuggestionList ajax');
  } catch (error) {
    console.log('getSuggestionList error: ', error);
    return [];
  }
  console.log('getSuggestionList response: ', response);
  const transformedSuggestionCollection = [];
  const textArr = response?.data?.[1];
  const descriptionArr = response?.data?.[2];
  const linkArr = response?.data?.[3];
  for (let id = 0; id < textArr.length; id += 1) {
    transformedSuggestionCollection.push({
      id,
      text: textArr?.[id] ?? '',
      description: descriptionArr?.[id] ?? '',
      link: linkArr?.[id] ?? '',
    });
  }
  await new Promise((resolve) => setTimeout(resolve, 2000));
  this.suggestionList = transformedSuggestionCollection;
  return transformedSuggestionCollection;
}

function handleSelectedSuggestion(selection) {
  clearTimeout(this.choiceWatcherDebouncedTimeoutInstance);
  this.selection = selection;
  // console.log('selection: ', selection);
}

function handleNonSuggestedChoice() {
  if (this.userEnteredText.length === 0) {
    return;
  }
  if (this.showLoading === true) {
    return;
  }
  console.log('handleNonSuggestedChoice userEnteredText: ', this.userEnteredText);
}

function choiceWatcherDebounced() {
  clearTimeout(this.choiceWatcherDebouncedTimeoutInstance);
  this.choiceWatcherDebouncedTimeoutInstance = setTimeout(this.handleNonSuggestedChoice, 600);
}

export default {
  name: 'landing-page',
  components: {
    VueSimpleSuggest,
  },
  data() {
    return {
      userEnteredText: '',
      autoCompleteStyle: {
        vueSimpleSuggest: 'position-relative',
        defaultInput: 'form-control',
        suggestions: 'position-absolute list-group z-1',
        suggestItem: 'list-group-item cursor-pointer',
      },
      showLoading: false,
      suggestionList: [],
      suggestionListIsOpen: false,
      choiceWatcherDebouncedTimeoutInstance: setTimeout(() => {}, 0),
      selection: {},
    };
  },
  watch: {
    userEnteredText: choiceWatcherDebounced,
  },
  methods: {
    getSuggestionList,
    // sleep,
    handleNonSuggestedChoice,
    handleSelectedSuggestion,
  },
};
</script>

<style>
  .list-group-item.hover {
    z-index: 2;
    color: #fff;
    background-color: #0d6efd;
    border-color: #0d6efd;
  }
  .z-1 {
    z-index: 1;
  }
  .cursor-pointer {
    cursor: pointer;
  }
</style>

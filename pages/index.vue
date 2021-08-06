<template>
  <div class="container">
    <div class="row">
      <div class="col mt-5">
        <h1 class="title">vue select</h1>

        <!-- <v-select :options="['foo', 'bar', 'baz', 'fud']"></v-select> -->
        <v-select
          placeholder="Find wikipedia Article"
          :filterable="false"
          :options="options"
          @search="handleOnSearchDebounced"
          :getOptionLabel="option => option.text ? option.text : ''"
          v-model="selected"
        ></v-select>

      </div>
    </div>
    <div class="row">
      <div class="col mt-5">
        <div v-if="selected && selected.text && selected.text.length > 0">
          <p>title: {{ selected.text }}</p>
          <!-- <p>description: {{ selected.description }}</p> -->
          <p>link: <a :href="selected.link" target="_blank" rel="nofollow">{{ selected.link }}</a></p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
function handleOnSearchDebounced(search, loading) {
  if (search.length === 0) {
    return;
  }
  loading(true);
  clearTimeout(this.searchDebouncedTimeoutInstance);
  this.searchDebouncedTimeoutInstance = setTimeout(() => this.getWikipediaSearchInfo(search, loading), 300);
}

async function getWikipediaSearchInfo(search, loading) {
  console.time('getWikipediaSearchInfo ajax');
  loading(true);
  const limit = 25;
  let response;
  try {
    response = await this.$axios({
      method: 'GET',
      url: `https://en.wikipedia.org/w/api.php?origin=*&action=opensearch&namespace=*&search=${search}&limit=${limit}&namespace=0&format=json`,
    });
    console.timeEnd('getWikipediaSearchInfo ajax');
  } catch (error) {
    console.log('getWikipediaSearchInfo error: ', error);
  }
  console.log('getWikipediaSearchInfo response: ', response);
  // await new Promise((resolve) => setTimeout(resolve, 2000)); // for testing only
  const transformedSuggestionCollection = [];
  const textArr = response?.data?.[1];
  const descriptionArr = response?.data?.[2];
  const linkArr = response?.data?.[3];
  for (let j = 0; j < textArr.length; j += 1) {
    transformedSuggestionCollection.push({
      text: textArr?.[j] ?? '',
      description: descriptionArr?.[j] ?? '',
      link: linkArr?.[j] ?? '',
    });
  }
  this.options = transformedSuggestionCollection;
  loading(false);
}

export default {
  name: 'landing-page',
  data() {
    return {
      options: [],
      searchDebouncedTimeoutInstance: setTimeout(() => {}, 0),
      selected: null,
    };
  },
  methods: {
    handleOnSearchDebounced,
    getWikipediaSearchInfo,
  },
};
</script>

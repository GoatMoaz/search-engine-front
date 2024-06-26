<template>
  <main class="min-h-screen bg-gray-800 flex justify-center items-center flex-col gap-10">
    <div class="flex flex-col bg-[#2f3640] w-2/4 rounded-t-3xl">
      <div class="searchBox">
        <input class="searchInput border-b border-gray-400" type="text" name="" placeholder="Search something"
          v-model="value" @keyup.enter="searchValueChange" :dir="direction" />
        <button class="searchButton" :class="{'right-2' : direction === 'ltr', 'left-2': direction === 'rtl'}" @click="searchValueChange">
          <FileSearchOutlined style="font-size: 24px" />
        </button>
      </div>
      <div>
        <transition-group name="fade" tag="div" class="max-h-80 flex flex-col w-full overflow-y-auto">
          <a :href="item" v-for="(item, index) in paginatedContent" :key="index"
            class="text-white p-6 border-b-2 border-gray-700 text-left hover:bg-gray-700 flex items-center gap-4 truncate">
            <SearchOutlined />
            {{ item }}
          </a>
        </transition-group>
      </div>
    </div>
    <a-pagination class="custom-pagination" :current="currentPage" :total="totalItems" :pageSize="itemsPerPage"
      v-if="fakeContent.length > 0"
      @change="handlePageChange" />

    <div class="flex space-x-2 border-[3px] border-[#1DDAB7] rounded-xl select-none">
      <label class="radio flex flex-grow items-center justify-center rounded-lg p-1 cursor-pointer">
        <input type="radio" name="radio" value="tfidf" class="peer hidden" checked="" v-model="search_type" />
        <span
          class="tracking-widest peer-checked:bg-gradient-to-r peer-checked:from-[#2af598] peer-checked:to-[#009efd] peer-checked:text-white text-white font-light p-2 rounded-lg">TF-IDF</span>
      </label>

      <label class="radio flex flex-grow items-center justify-center rounded-lg p-1 cursor-pointer">
        <input v-model="search_type" type="radio" name="radio" value="pageRank" class="peer hidden" />
        <span
          class="tracking-widest peer-checked:bg-gradient-to-r peer-checked:from-[#2af598] peer-checked:to-[#009efd] peer-checked:text-white text-white font-light p-2 rounded-lg">
          Page Rank
        </span>
      </label>
    </div>
  </main>
</template>

<script setup>
import { ref, computed } from 'vue';
const value = ref("");
const search_type = ref("tfidf");
const fakeContent = ref([]);
const currentPage = ref(1);
const itemsPerPage = ref(10);
const totalItems = computed(() => fakeContent.value.length);

watch(search_type, () => {
  searchValueChange();
});

const direction = computed(() => {
  const arabic_regex = /[\u0600-\u06FF]/;
  return arabic_regex.test(value.value) ? "rtl" : "ltr";
})

const searchValueChange = async () => {
  if (search_type.value === "tfidf") {
    const { data } = await useFetch(`/api/tf-idf?value=${value.value}`);
    fakeContent.value = data.value.results;
  } else {
    const { data } = await useFetch(`/api/page_rank?value=${value.value}`);
    fakeContent.value = data.value.results;
  }
  currentPage.value = 1; // Reset to first page when new search is made

};

const handlePageChange = (page) => {
  currentPage.value = page;
};
const paginatedContent = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage.value;
  const end = start + itemsPerPage.value;
  return fakeContent.value.slice(start, end);
});
</script>

<style>

.ant-pagination .ant-pagination-item-active {
  background-color: transparent !important;
  border-color: #1DDAB7 !important;
  border-radius: 50% !important;
}

.ant-select-selector {
  background-color: #2f3640 !important;
  border: 1px solid #2f3640 !important;
}

.ant-pagination-options{
   display: none !important;
}

.ant-pagination .ant-pagination-item a,
.ant-pagination-item-ellipsis,
.ant-pagination .ant-pagination-next button,
.ant-pagination .ant-pagination-prev button,
.ant-select-selection-item {
  color: white !important;
}

.ant-pagination-disabled {
  color: rgba(255, 255, 255, 0.268) !important;
}

/* The emerging W3C standard that is currently Firefox only */
* {
  scrollbar-width: thin;
  scrollbar-color: rgba(155, 155, 155, 0.7) rgba(132, 124, 124, 0);
}

/* Works on Chrome/Edge/Safari */
*::-webkit-scrollbar {
  width: 12px;
}

*::-webkit-scrollbar-track {
  background: rgba(12, 12, 12, 0.085);
}

*::-webkit-scrollbar-thumb {
  background-color: rgba(155, 155, 155, 0.7);
  border-radius: 20px;
  border: 3px solid rgba(0, 0, 0, 0.146);
}

.searchBox {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
  background: #2f3640;
  border-radius: 50px;
  position: relative;
}

.searchButton {
  color: white;
  position: absolute;
  right: 8px;
  width: 50px;
  height: 50px;
  border-radius: 50%;
  background: var(--gradient-2, linear-gradient(90deg, #2af598 0%, #009efd 100%));
  border: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 300ms cubic-bezier(0.23, 1, 0.32, 1);
}

/*hover effect*/
button:hover {
  color: #fff;
  background-color: #1a1a1a;
  box-shadow: rgba(0, 0, 0, 0.5) 0 10px 20px;
  transform: translateY(-3px);
}

/*button pressing effect*/
button:active {
  box-shadow: none;
  transform: translateY(0);
}

.searchInput {
  border: none;
  background: none;
  outline: none;
  color: white;
  font-size: 15px;
  padding: 24px 46px 24px 26px;
  width: 100%;
}
</style>

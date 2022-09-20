<script setup>
import Paper from './Paper.vue'
import PaperIcon from './icons/PaperIcon.vue'
import { reactive } from 'vue'

import arxivs from '@/assets/papers/arxivs.yml'

const papers = reactive([])

// @@@@ arxiv @@@@@
const arxivPromise = new Promise((resolve, reject) => {
  console.log(`https://export.arxiv.org/api/query?id_list=${arxivs.join()}&max_results=100`)
  fetch(`https://export.arxiv.org/api/query?id_list=${arxivs.join()}&max_results=100`)
    .then(response => response.text())
    .then(xml => new DOMParser().parseFromString(xml, 'text/xml'))
    .then(data => {
      const entries = data.getElementsByTagName('entry')
      for (let i = 0; i < entries.length; i++) {
        const entry = entries[i]
        /* console.log(entry) */
        const getText = (el, tag) => el.getElementsByTagName(tag)[0].textContent
        papers.push({
          title: getText(entry, 'title'),
          abstract: getText(entry, 'summary'),
          pdf: entry.querySelector('link[title="pdf"]').getAttribute('href'),
          authors: Array.from(entry.getElementsByTagName('author'))
            .map(author => getText(author, 'name')),
          publisheDate: new Date(Date.parse(getText(entry, 'published'))),
          showDetail: false,
        })
      }
      resolve()
    })
    .catch(e => reject(e))
})

Promise.all([ arxivPromise ])
  .then(() => {
    // latest first
    papers.sort((a, b) =>  b.publisheDate.getTime() - a.publisheDate.getTime())
  })
</script>

<template>
  <Paper v-for="paper in papers">
    <template #icon>
      <PaperIcon />
    </template>
    <template #heading>{{ paper.title }}</template>
    {{ paper.publisheDate.toDateString() }}<br>
    <template v-for="author in paper.authors">
      <template v-if="author !== paper.authors[0]">, </template>
      <a>{{ author }}</a>
    </template><br>

    <template v-if="paper.showDetail">
      <span class="clickable" @click="paper.showDetail = false">📖 Hide detail</span>
      <br>
      {{ paper.abstract }}
      <br>
      [<a :href="paper.pdf" target="_blank" rel="noopener">pdf</a>]
    </template>
    <template v-else>
      <span class="clickable" @click="paper.showDetail = true">📖 Show detail</span>
    </template>
  </Paper>

</template>

<style scoped>
span.clickable {
  transition: 0.4s;
}
span.clickable:hover {
  cursor: pointer;
  background-color: hsla(210, 100%, 40%, 0.2);
}
</style>

<script setup lang="ts">
import type { DefaultTheme } from 'vitepress/theme'
import docsearch from '@docsearch/js'
import { onMounted, watch } from 'vue'
import { useRouter, useRoute } from 'vitepress'
import { useData } from '../composables/data'

const props = defineProps<{
  algolia: DefaultTheme.AlgoliaSearchOptions
}>()

const router = useRouter()
const route = useRoute()
const { site, localeIndex, lang } = useData()

onMounted(update)
watch(localeIndex, update)

function update() {
  const options = {
    ...props.algolia,
    ...props.algolia.locales?.[localeIndex.value]
  }
  const rawFacetFilters = options.searchParameters?.facetFilters ?? []
  const facetFilters = [
    ...(Array.isArray(rawFacetFilters)
      ? rawFacetFilters
      : [rawFacetFilters]
    ).filter((f) => !f.startsWith('lang:')),
    `lang:${lang.value}`
  ]
  initialize({
    ...options,
    searchParameters: {
      ...options.searchParameters,
      facetFilters
    }
  })
}

function initialize(userOptions: DefaultTheme.AlgoliaSearchOptions) {
  const options = Object.assign({}, userOptions, {
    container: '#docsearch',

    navigator: {
      navigate({ itemUrl }: { itemUrl: string }) {
        const { pathname: hitPathname } = new URL(
          window.location.origin + itemUrl
        )

        // router doesn't handle same-page navigation so we use the native
        // browser location API for anchor navigation
        if (route.path === hitPathname) {
          window.location.assign(window.location.origin + itemUrl)
        } else {
          router.go(itemUrl)
        }
      }
    },

    transformItems(items: any[]) {
      return items.map((item: any) => {
        return Object.assign({}, item, {
          url: getRelativePath(item.url)
        })
      })
    },

    hitComponent({ hit, children }: { hit: any; children: any }) {
      return {
        __v: null,
        type: 'a',
        ref: undefined,
        constructor: undefined,
        key: undefined,
        props: { href: hit.url, children }
      }
    }
  } as any)

  docsearch(options)
}

function getRelativePath(absoluteUrl: string) {
  const { pathname, hash } = new URL(absoluteUrl)
  return (
    pathname.replace(
      /\.html$/,
      site.value.cleanUrls ? '' : '.html'
    ) + hash
  )
}
</script>

<template>
  <div id="docsearch" />
</template>

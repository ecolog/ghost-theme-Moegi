<script>
  import { onMount, createEventDispatcher } from 'svelte';
  import Hero from "../components/Hero.svelte";
  import Pagination from "../components/Pagination.svelte";
  //Feed Item
  import Note from "../components/FeedItem/Note.svelte";
  import Pure from "../components/FeedItem/Pure.svelte";

  export let params = {};

  const api = window.ghostAPI;
  const dispatch = createEventDispatcher();
  let postList = [];
  let paginationData = {};
  let page = params.page || 1;
  let tagSlug = params.tag_slug || '';
  let authorSlug = params.author_slug || '';
  let feedType = config.feed ? config.feed.toLowerCase() : 'pure';

  onMount(async () => {
    let filter = ''
    if (tagSlug) {
      filter = 'tag:' + tagSlug
    } else if (authorSlug) {
      filter = 'author:' + authorSlug
    }
    postList = await api.posts.browse({ limit: 8, page, include: "tags", filter});
    paginationData = postList.meta.pagination;
    dispatch("message", {
      func: "setLoadStatus",
      data: {
        loaded: true
      }
    });
	});

  const getPostIndex = index => {
    return paginationData.total - (paginationData.page - 1) * paginationData.limit - index;
  };
</script>

<style lang="scss">
.filter-container {
  margin-bottom: 16px;
  .filter {
    display: flex;
    align-items: center;
    font-size: 0.9em;
    padding: 8px 12px;
    color: var(--color-text);
    background-color: var(--color-decoration);
    border-radius: 4px;
    .iconfont {
      font-size: 1.1em;
      margin-right: 6px;
    }
    .text {
      flex: 1;
    }
    .total {
      margin-right: 4px;
    }
  }
}
</style>

<template>
  <Hero/>
  <div class="gh-container bg-white">
    {#if tagSlug || authorSlug}
      <div class="filter-container">
        <div class="filter">
          {#if tagSlug}
            <span class="iconfont icon-tags"></span>
          {:else if authorSlug}
            <span class="iconfont icon-user"></span>
          {/if}
          <span class="text">{tagSlug || authorSlug}</span>
          <span class="total">{paginationData.total}</span>
        </div>
      </div>
    {/if}
    <div class="feed-list simple">
      {#each postList as postItem, i}
        {#if feedType === 'note'}
          <Note data={postItem} />
        {:else if feedType === 'pure'}
          <Pure data={postItem} index={getPostIndex(i)} showIndex={!tagSlug && !authorSlug} />
        {:else}
          <Pure data={postItem} index={getPostIndex(i)} showIndex={!tagSlug && !authorSlug} />
        {/if}
      {/each}
    </div>
    {#if paginationData}
      <Pagination {paginationData} />
    {/if}
  </div>
</template>

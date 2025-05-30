
<script lang="ts">
  import Paper from '@smui/paper';
  import { onDestroy } from 'svelte';

  import ModelBreadcrumb from './ModelBreadcrumb.svelte'

  import type { Model } from '@/types/models';
  import { modelsSelected, modelsSelectedLimit, errorMessage } from '@/store';
  
  export let modelItem: Model;
  let modelIsSelected: boolean = false;
  const limit = $modelsSelectedLimit;

  const unsubscribeModels = modelsSelected.subscribe(() => {
    modelIsSelected = $modelsSelected.some(
      model => model.name.includes(modelItem.name)
    )
  });

  function toggleModel(modelItem: Model) {
    if (modelIsSelected) {
      $modelsSelected = $modelsSelected.filter(
        model => model.name !== modelItem.name
      );
    } else {
      if (limit === 1) {
        $modelsSelected = [modelItem];
        return;
      }
      if (limit > 1 && $modelsSelected.length >= limit) {
        errorMessage.set('Models selected reached limit');
        return;
      }
      $modelsSelected = [...$modelsSelected, modelItem];
    }
  }

  function ucClick(modelItem: Model) {
    toggleModel(modelItem);
  }

  function sanitize(rawHtml: string) {
    return rawHtml.replaceAll('\\_', '_');
  }

  onDestroy(unsubscribeModels);
</script>



<div class="models-card-container">
    
  <div class="card-container { modelIsSelected ? 'is-selected' : '' }">
    <Paper elevation="5" on:click={() => ucClick(modelItem)}>

      <div class="grid-container">
        <div class="breadcrumbs">
          <ModelBreadcrumb modelItem={ modelItem } />
        </div>
        
        <div class="img-container">
          {#each modelItem.images as image}
            <div class="image">
              <img
                loading="lazy"
                src={ image.url }
                alt={ image.caption }
              >
            </div>
          {/each}
        </div>

        <div class="text-content">
          <div class="description">{ @html sanitize(modelItem.description) }</div>
          
          {#if modelItem.author.length}
            <div class="contributors-container">
              <span class="credit-title">Credits:</span>
              {#each modelItem.author as author}
                <span class="contributor-names">{ author.given_name } { author.family_name }</span>
              {/each}
            </div>
          {/if}
        </div>
      </div>

    </Paper>
  </div> <!-- end card -->
</div>



<style>
  .models-card-container {
    margin-bottom: 10px;
  }
  .models-card-container .grid-container {
    display: grid;
    grid-template:
      "title title" auto
      "imgs text" auto
      / 1fr 1fr;
  }
  .models-card-container .breadcrumbs {
    grid-area: title;
  }
  .models-card-container .text-content {
    grid-area: text;
  }
  .models-card-container .img-container {
    grid-area: imgs;
  }
 
  .img-container {
    display: flex;
    align-items: center;
    justify-content: space-around;
  }
  .image img {
    width: 215px;
    margin-right: 15px;
  }

  .text-content {
    display: flex;
    flex-direction: column;
    justify-content: space-between;
  }
  .description {
    font-size: 1em;
  }
  .contributors-container {
    font-size: 0.8em;
  }
  .credit-title {
    font-weight: bold;
  }
  span.contributor-names:before {
    content: " - ";
  }

  :global(.card-container.is-selected div.smui-paper) {
    background-color: #e2eeff;
  }

  @media only screen and (max-width: 900px) {
    .models-card-container .grid-container {
      grid-template:
        "title" auto
        "imgs" auto
        "text" auto
        / 1fr;
    }
  }
</style>

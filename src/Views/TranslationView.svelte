<script lang="ts">
  import { onMount, onDestroy } from "svelte";

  import { targetWords } from "../Stores/stores";


  import { translate, GoogleTranslationRes } from "../API/google-translate";
  import DropdownMenu from "../Components/DropdownMenu.svelte";


  onDestroy(async () => {
    unsubscribe();
  });

  const unsubscribe = targetWords.subscribe(value => {
		translateWords(null);
	});

  let languages = ["Turkish", "English"];

  let translationResult: GoogleTranslationRes;

  let showSimilarWordsForTranslations = false;

  async function translateWords(e) {

    let result = await translate($targetWords, { from: "en", to: "tr" }, false);

    if (result) {
      if (
        result.hasOwnProperty("translations") ||
        result.hasOwnProperty("translation")
      ) {
        translationResult = result;
      }
    }
    console.log(translationResult);
  }
</script>

<style lang="scss">
  textarea {
    color: white;
    background-color: rgb(51, 51, 51);
    border: none;

    &::placeholder {
      color: rgb(167, 167, 167);
    }
  }

  top-bar {
    display: grid;
    grid-auto-flow: column;
    grid-auto-columns: 1fr;
    button {
      justify-self: center;
      width: max-content;
    }
  }

  .translate-from {
    justify-self: right;
  }

  #translations-of {
    color: white;
    padding: 5px;
    background-color: rgb(41, 41, 41);
  }

  .translate-result-area {
    display: grid;
    row-gap: 5px;
    grid-template-columns: max-content;
    color: rgb(167, 167, 167);
    background-color: rgb(66, 66, 66);

    padding-bottom: 6px;

    .translate-result-area-header {
      color: rgb(255, 221, 87);
    }

    .translate-result-line {
      display: grid;
      grid-auto-flow: column;
      grid-auto-columns: 1fr;

      padding: 3px;

      .translation-word-of-line {
        color: white;
      }
    }
  }
</style>

<top-bar>
  <DropdownMenu placeholder="Translate To" items={languages} />
  <button class="button is-link">
    <span class="icon is-medium"> <i class="fas fa-exchange-alt" /> </span>
  </button>
  <DropdownMenu
    justifySelfRight={true}
    placeholder="Translate From"
    items={languages} />
</top-bar>

<textarea
  bind:value={$targetWords}
  on:input={translateWords}
  class="textarea has-fixed-size"
  placeholder="Translate"
  name="main-text-area"
  id="main-text-area"
  cols={30}
  rows={5} />
{#if translationResult != null}
  {#if translationResult.target != null}
    <div id="translations-of">Translations of {$targetWords}</div>
  {/if}
  <div class="translate-result-area">
    <span class="translate-result-area-header" style="font-weight: bold;">
      most common
    </span>
    <span style="font-weight: bold; color:white;">
      {translationResult.translation}
    </span>
  </div>
  {#if translationResult.translations}
    {#each translationResult.translations as translation}
      <div class="translate-result-area">
        <span class="translate-result-area-header">{translation.type}</span>
        {#each translation.content as line}
          <div class="translate-result-line">
            <div>
              <div class="frequency-bar" />
              <span class="translation-word-of-line">{line.word}</span>
            </div>
            {#if showSimilarWordsForTranslations}
              <div>{line.meaning.join(', ')}</div>
            {/if}
          </div>
        {/each}
      </div>
    {/each}
  {/if}
{/if}

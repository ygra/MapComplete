<script lang="ts">
  import type { Feature } from "geojson"
  import { UIEventSource } from "../../Logic/UIEventSource"
  import LayerConfig from "../../Models/ThemeConfig/LayerConfig"
  import type { SpecialVisualizationState } from "../SpecialVisualization"
  import TagRenderingEditable from "../Popup/TagRendering/TagRenderingEditable.svelte"
  import { onDestroy } from "svelte"
  import Translations from "../i18n/Translations"
  import Tr from "../Base/Tr.svelte"

  export let state: SpecialVisualizationState
  export let layer: LayerConfig
  export let selectedElement: Feature
  export let tags: UIEventSource<Record<string, string>>
  export let highlightedRendering: UIEventSource<string> = undefined

  let _metatags: Record<string, string>
  onDestroy(
    state.userRelatedState.preferencesAsTags.addCallbackAndRun((tags) => {
      _metatags = tags
    })
  )
</script>

{#if $tags._deleted === "yes"}
  <Tr t={Translations.t.delete.isDeleted} />
  <button class="w-full" on:click={() => state.selectedElement.setData(undefined)}>
    <Tr t={Translations.t.general.returnToTheMap} />
  </button>
{:else}
  <div class="flex flex-col gap-y-2 overflow-y-auto p-1 px-2">
    {#each layer.tagRenderings as config (config.id)}
      {#if (config.condition?.matchesProperties($tags) ?? true) && config.metacondition?.matchesProperties({ ...$tags, ..._metatags } ?? true)}
        {#if config.IsKnown($tags)}
          <TagRenderingEditable
            {tags}
            {config}
            {state}
            {selectedElement}
            {layer}
            {highlightedRendering}
          />
        {/if}
      {/if}
    {/each}
  </div>
{/if}

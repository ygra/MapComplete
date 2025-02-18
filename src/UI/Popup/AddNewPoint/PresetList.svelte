<script lang="ts">
  import LayoutConfig from "../../../Models/ThemeConfig/LayoutConfig"
  import { createEventDispatcher } from "svelte"
  import type PresetConfig from "../../../Models/ThemeConfig/PresetConfig"
  import Tr from "../../Base/Tr.svelte"
  import Translations from "../../i18n/Translations.js"
  import { Translation } from "../../i18n/Translation"
  import type { SpecialVisualizationState } from "../../SpecialVisualization"
  import { ImmutableStore } from "../../../Logic/UIEventSource"
  import { TagUtils } from "../../../Logic/Tags/TagUtils"
  import LayerConfig from "../../../Models/ThemeConfig/LayerConfig"
  import FromHtml from "../../Base/FromHtml.svelte"
  import NextButton from "../../Base/NextButton.svelte"

  /**
   * This component lists all the presets and allows the user to select one
   */
  export let state: SpecialVisualizationState
  let layout: LayoutConfig = state.layout
  let presets: {
    preset: PresetConfig
    layer: LayerConfig
    text: Translation
    icon: string
    tags: Record<string, string>
  }[] = []

  for (const layer of layout.layers) {
    const flayer = state.layerState.filteredLayers.get(layer.id)
    if (flayer.isDisplayed.data === false) {
      // The layer is not displayed...
      if (!state.featureSwitches.featureSwitchFilter.data) {
        // ...and we cannot enable the layer control -> we skip, as these presets can never be shown anyway
        continue
      }

      if (layer.name === undefined) {
        // this layer can never be toggled on in any case, so we skip the presets
        continue
      }
    }

    for (const preset of layer.presets) {
      const tags = TagUtils.KVtoProperties(preset.tags ?? [])

      const icon: string = layer.mapRendering[0]
        .RenderIcon(new ImmutableStore<any>(tags), false)
        .html.SetClass("w-12 h-12 block relative")
        .ConstructElement().innerHTML

      const description = preset.description?.FirstSentence()

      const simplified = {
        preset,
        layer,
        icon,
        description,
        tags,
        text: Translations.t.general.add.addNew.Subs(
          { category: preset.title },
          preset.title["context"]
        ),
      }
      presets.push(simplified)
    }
  }

  const dispatch = createEventDispatcher<{
    select: { preset: PresetConfig; layer: LayerConfig; icon: string; tags: Record<string, string> }
  }>()
</script>

<div class="flex w-full flex-col">
  <h2 class="mr-12">
    <!-- The title gets a big right margin to give place to the 'close'-button, see https://github.com/pietervdvn/MapComplete/issues/1445 -->
    <Tr t={Translations.t.general.add.intro} />
  </h2>

  {#each presets as preset}
    <NextButton on:click={() => dispatch("select", preset)}>
      <FromHtml slot="image" src={preset.icon} />
      <div class="flex flex-col">
        <b class="w-fit">
          <Tr t={preset.text} />
        </b>
        {#if preset.description}
          <Tr t={preset.description} cls="font-normal" />
        {/if}
      </div>
    </NextButton>
  {/each}
</div>

<script lang="ts">
  import FeatureReviews from "../../Logic/Web/MangroveReviews"
  import SingleReview from "./SingleReview.svelte"
  import { Utils } from "../../Utils"
  import StarsBar from "./StarsBar.svelte"
  import ReviewForm from "./ReviewForm.svelte"
  import Translations from "../i18n/Translations"
  import Tr from "../Base/Tr.svelte"
  import type { SpecialVisualizationState } from "../SpecialVisualization"
  import { UIEventSource } from "../../Logic/UIEventSource"
  import type { Feature } from "geojson"
  import LayerConfig from "../../Models/ThemeConfig/LayerConfig"
  import ToSvelte from "../Base/ToSvelte.svelte"
  import Svg from "../../Svg"

  /**
   * An element showing all reviews
   */
  export let reviews: FeatureReviews
  export let state: SpecialVisualizationState
  export let tags: UIEventSource<Record<string, string>>
  export let feature: Feature
  export let layer: LayerConfig
  let average = reviews.average
  let _reviews = []
  reviews.reviews.addCallbackAndRunD((r) => {
    _reviews = Utils.NoNull(r)
  })
</script>

<div class="border-2 border-dashed border-gray-300">
  {#if _reviews.length > 1}
    <StarsBar score={$average} />
  {/if}
  {#if _reviews.length > 0}
    {#each _reviews as review}
      <SingleReview {review} />
    {/each}
  {:else}
    <Tr t={Translations.t.reviews.no_reviews_yet} />
  {/if}
  <div class="flex justify-end">
    <ToSvelte construct={Svg.mangrove_logo_svg().SetClass("w-12 h-12")} />
    <Tr t={Translations.t.reviews.attribution} />
  </div>
</div>

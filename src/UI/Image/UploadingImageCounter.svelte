<script lang="ts">
  /**
   * Shows information about how much images are uploaded for the given feature
   */

  import type { SpecialVisualizationState } from "../SpecialVisualization"
  import { Store } from "../../Logic/UIEventSource"
  import type { OsmTags } from "../../Models/OsmFeature"
  import Translations from "../i18n/Translations"
  import Tr from "../Base/Tr.svelte"
  import Loading from "../Base/Loading.svelte"

  export let state: SpecialVisualizationState
  export let tags: Store<OsmTags>
  const featureId = tags.data.id
  const { uploadStarted, uploadFinished, retried, failed } =
    state.imageUploadManager.getCountsFor(featureId)
  const t = Translations.t.image
</script>

{#if $uploadStarted == 1}
  {#if $uploadFinished == 1}
    <Tr cls="thanks" t={t.upload.one.done} />
  {:else if $failed == 1}
    <div class="alert flex flex-col">
      <Tr cls="self-center" t={t.upload.one.failed} />
      <Tr t={t.upload.failReasons} />
      <Tr t={t.upload.failReasonsAdvanced} />
    </div>
  {:else if $retried == 1}
    <Loading cls="alert">
      <Tr t={t.upload.one.retrying} />
    </Loading>
  {:else}
    <Loading cls="alert">
      <Tr t={t.upload.one.uploading} />
    </Loading>
  {/if}
{:else if $uploadStarted > 1}
  {#if $uploadFinished + $failed == $uploadStarted && $uploadFinished > 0}
    <Tr cls="thanks" t={t.upload.multiple.done.Subs({ count: $uploadFinished })} />
  {:else if $uploadFinished == 0}
    <Loading cls="alert">
      <Tr t={t.upload.multiple.uploading.Subs({ count: $uploadStarted })} />
    </Loading>
  {:else if $uploadFinished > 0}
    <Loading cls="alert">
      <Tr
        t={t.upload.multiple.partiallyDone.Subs({
          count: $uploadStarted - $uploadFinished,
          done: $uploadFinished,
        })}
      />
    </Loading>
  {/if}
  {#if $failed > 0}
    <div class="alert flex flex-col">
      {#if failed === 1}
        <Tr cls="self-center" t={t.upload.one.failed} />
      {:else}
        <Tr cls="self-center" t={t.upload.multiple.someFailed.Subs({ count: $failed })} />
      {/if}
      <Tr t={t.upload.failReasons} />
      <Tr t={t.upload.failReasonsAdvanced} />
    </div>
  {/if}
{/if}

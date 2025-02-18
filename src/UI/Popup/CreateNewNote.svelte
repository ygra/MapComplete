<script lang="ts">
  /**
   * UIcomponent to create a new note at the given location
   */
  import type { SpecialVisualizationState } from "../SpecialVisualization"
  import { UIEventSource } from "../../Logic/UIEventSource"
  import { LocalStorageSource } from "../../Logic/Web/LocalStorageSource"
  import ValidatedInput from "../InputElement/ValidatedInput.svelte"
  import SubtleButton from "../Base/SubtleButton.svelte"
  import Tr from "../Base/Tr.svelte"
  import Translations from "../i18n/Translations.js"
  import type { Feature, Point } from "geojson"
  import LoginToggle from "../Base/LoginToggle.svelte"
  import FilteredLayer from "../../Models/FilteredLayer"

  export let coordinate: { lon: number; lat: number }
  export let state: SpecialVisualizationState

  let comment: UIEventSource<string> = LocalStorageSource.Get("note-text")
  let created = false

  let notelayer: FilteredLayer = state.layerState.filteredLayers.get("note")

  let hasFilter = notelayer?.hasFilter
  let isDisplayed = notelayer?.isDisplayed

  function enableNoteLayer() {
    state.guistate.closeAll()
    isDisplayed.setData(true)
  }

  async function uploadNote() {
    let txt = comment.data
    if (txt === undefined || txt === "") {
      return
    }
    const loc = coordinate
    txt += "\n\n #MapComplete #" + state?.layout?.id
    const id = await state?.osmConnection?.openNote(loc.lat, loc.lon, txt)
    console.log("Created a note, got id", id)
    const feature = <Feature<Point>>{
      type: "Feature",
      geometry: {
        type: "Point",
        coordinates: [loc.lon, loc.lat],
      },
      properties: {
        id: "" + id.id,
        date_created: new Date().toISOString(),
        _first_comment: txt,
        comments: JSON.stringify([
          {
            text: txt,
            html: txt,
            user: state.osmConnection?.userDetails?.data?.name,
            uid: state.osmConnection?.userDetails?.data?.uid,
          },
        ]),
      },
    }
    // Normally, the 'Changes' will generate the new element. The 'notes' are an exception to this
    state.newFeatures.features.data.push(feature)
    state.newFeatures.features.ping()
    state.selectedElement?.setData(feature)
    if (state.featureProperties.trackFeature) {
      state.featureProperties.trackFeature(feature)
    }
    comment.setData("")
    created = true
    state.selectedElement.setData(feature)
    state.selectedLayer.setData(state.layerState.filteredLayers.get("note"))
  }
</script>

{#if notelayer === undefined}
  <div class="alert">
    This theme does not include the layer 'note'. As a result, no nodes can be created
  </div>
{:else if created}
  <div class="thanks">
    <Tr t={Translations.t.notes.isCreated} />
  </div>
{:else}
  <h3>
    <Tr t={Translations.t.notes.createNoteTitle} />
  </h3>

  {#if $isDisplayed}
    <!-- The layer is displayed, so we can add a note without worrying for duplicates -->
    {#if $hasFilter}
      <div class="flex flex-col">
        <!-- ...but a filter is set ...-->
        <div class="alert">
          <Tr t={Translations.t.notes.noteLayerHasFilters} />
        </div>
        <SubtleButton on:click={() => notelayer.disableAllFilters()}>
          <img slot="image" src="./assets/svg/filter.svg" class="mr-4 h-8 w-8" />
          <Tr slot="message" t={Translations.t.notes.disableAllNoteFilters} />
        </SubtleButton>
      </div>
    {:else}
      <div>
        <Tr t={Translations.t.notes.createNoteIntro} />
        <div class="border-grey-500 rounded-sm border">
          <div class="w-full p-1">
            <ValidatedInput type="text" value={comment} />
          </div>

          <LoginToggle {state}>
            <span slot="loading"><!--empty: don't show a loading message--></span>
            <div slot="not-logged-in" class="alert">
              <Tr t={Translations.t.notes.warnAnonymous} />
            </div>
          </LoginToggle>

          {#if $comment?.length >= 3}
            <SubtleButton on:click={uploadNote}>
              <img slot="image" src="./assets/svg/addSmall.svg" class="mr-4 h-8 w-8" />
              <Tr slot="message" t={Translations.t.notes.createNote} />
            </SubtleButton>
          {:else}
            <div class="alert">
              <Tr t={Translations.t.notes.textNeeded} />
            </div>
          {/if}
        </div>
      </div>
    {/if}
  {:else}
    <div class="flex flex-col">
      <div class="alert">
        <Tr t={Translations.t.notes.noteLayerNotEnabled} />
      </div>
      <SubtleButton on:click={enableNoteLayer}>
        <img slot="image" src="./assets/svg/layers.svg" class="mr-4 h-8 w-8" />
        <Tr slot="message" t={Translations.t.notes.noteLayerDoEnable} />
      </SubtleButton>
    </div>
  {/if}
{/if}

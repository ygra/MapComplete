<script lang="ts">
  /**
   * Shows an 'upload'-button which will start the upload for this feature
   */

  import type { SpecialVisualizationState } from "../SpecialVisualization"
  import { Store } from "../../Logic/UIEventSource"
  import type { OsmTags } from "../../Models/OsmFeature"
  import LoginToggle from "../Base/LoginToggle.svelte"
  import Translations from "../i18n/Translations"
  import Tr from "../Base/Tr.svelte"
  import UploadingImageCounter from "./UploadingImageCounter.svelte"
  import FileSelector from "../Base/FileSelector.svelte"
  import ToSvelte from "../Base/ToSvelte.svelte"
  import Svg from "../../Svg"

  export let state: SpecialVisualizationState

  export let tags: Store<OsmTags>
  /**
   * Image to show in the button
   * NOT the image to upload!
   */
  export let image: string = undefined
  if (image === "") {
    image = undefined
  }
  export let labelText: string = undefined
  const t = Translations.t.image

  let licenseStore = state.userRelatedState.imageLicense

  function handleFiles(files: FileList) {
    for (let i = 0; i < files.length; i++) {
      const file = files.item(i)
      console.log("Got file", file.name)
      try {
        state.imageUploadManager.uploadImageAndApply(file, tags)
      } catch (e) {
        alert(e)
      }
    }
  }
</script>

<LoginToggle {state}>
  <Tr slot="not-logged-in" t={t.pleaseLogin} />
  <div class="flex flex-col">
    <UploadingImageCounter {state} {tags} />
    <FileSelector
      accept="image/*"
      cls="button border-2 text-2xl"
      multiple={true}
      on:submit={(e) => handleFiles(e.detail)}
    >
      <div class="flex items-center">
        {#if image !== undefined}
          <img src={image} />
        {:else}
          <ToSvelte construct={Svg.camera_plus_svg().SetClass("block w-12 h-12 p-1 text-4xl ")} />
        {/if}
        {#if labelText}
          {labelText}
        {:else}
          <Tr t={t.addPicture} />
        {/if}
      </div>
    </FileSelector>
    <div class="text-sm">
      <Tr t={t.respectPrivacy} />
      <a
        class="cursor-pointer"
        on:click={() => {
          state.guistate.openUsersettings("picture-license")
        }}
      >
        <Tr t={t.currentLicense.Subs({ license: $licenseStore })} />
      </a>
    </div>
  </div>
</LoginToggle>

<script lang="ts">
    import { Store, UIEventSource } from "../Logic/UIEventSource";
    import { Map as MlMap } from "maplibre-gl";
    import MaplibreMap from "./Map/MaplibreMap.svelte";
    import FeatureSwitchState from "../Logic/State/FeatureSwitchState";
    import MapControlButton from "./Base/MapControlButton.svelte";
    import ToSvelte from "./Base/ToSvelte.svelte";
    import If from "./Base/If.svelte";
    import { GeolocationControl } from "./BigComponents/GeolocationControl";
    import type { Feature } from "geojson";
    import SelectedElementView from "./BigComponents/SelectedElementView.svelte";
    import LayerConfig from "../Models/ThemeConfig/LayerConfig";
    import Filterview from "./BigComponents/Filterview.svelte";
    import ThemeViewState from "../Models/ThemeViewState";
    import type { MapProperties } from "../Models/MapProperties";
    import Geosearch from "./BigComponents/Geosearch.svelte";
    import Translations from "./i18n/Translations";
    import { CogIcon, EyeIcon, MenuIcon, XCircleIcon } from "@rgossiaux/svelte-heroicons/solid";

    import Tr from "./Base/Tr.svelte";
    import CommunityIndexView from "./BigComponents/CommunityIndexView.svelte";
    import FloatOver from "./Base/FloatOver.svelte";
    import PrivacyPolicy from "./BigComponents/PrivacyPolicy";
    import Constants from "../Models/Constants";
    import TabbedGroup from "./Base/TabbedGroup.svelte";
    import UserRelatedState from "../Logic/State/UserRelatedState";
    import LoginToggle from "./Base/LoginToggle.svelte";
    import LoginButton from "./Base/LoginButton.svelte";
    import CopyrightPanel from "./BigComponents/CopyrightPanel";
    import DownloadPanel from "./DownloadFlow/DownloadPanel.svelte";
    import ModalRight from "./Base/ModalRight.svelte";
    import { Utils } from "../Utils";
    import Hotkeys from "./Base/Hotkeys";
    import { VariableUiElement } from "./Base/VariableUIElement";
    import SvelteUIElement from "./Base/SvelteUIElement";
    import OverlayToggle from "./BigComponents/OverlayToggle.svelte";
    import LevelSelector from "./BigComponents/LevelSelector.svelte";
    import ExtraLinkButton from "./BigComponents/ExtraLinkButton";
    import SelectedElementTitle from "./BigComponents/SelectedElementTitle.svelte";
    import Svg from "../Svg";
    import ThemeIntroPanel from "./BigComponents/ThemeIntroPanel.svelte";
    import type { RasterLayerPolygon } from "../Models/RasterLayers";
    import { AvailableRasterLayers } from "../Models/RasterLayers";
    import RasterLayerOverview from "./Map/RasterLayerOverview.svelte";
    import IfHidden from "./Base/IfHidden.svelte";
    import { onDestroy } from "svelte";
    import { OpenJosm } from "./BigComponents/OpenJosm";
    import MapillaryLink from "./BigComponents/MapillaryLink.svelte";
    import OpenIdEditor from "./BigComponents/OpenIdEditor.svelte";
    import OpenBackgroundSelectorButton from "./BigComponents/OpenBackgroundSelectorButton.svelte";
    import StateIndicator from "./BigComponents/StateIndicator.svelte";
    import LanguagePicker from "./LanguagePicker";
    import Locale from "./i18n/Locale";
    import ShareScreen from "./BigComponents/ShareScreen.svelte";

    export let state: ThemeViewState;
    let layout = state.layout;

    let maplibremap: UIEventSource<MlMap> = state.map;
    let selectedElement: UIEventSource<Feature> = state.selectedElement;
    let selectedLayer: UIEventSource<LayerConfig> = state.selectedLayer;

    const selectedElementView = selectedElement.map(
        (selectedElement) => {
            // Svelte doesn't properly reload some of the legacy UI-elements
            // As such, we _reconstruct_ the selectedElementView every time a new feature is selected
            // This is a bit wasteful, but until everything is a svelte-component, this should do the trick
            const layer = selectedLayer.data;
            if (selectedElement === undefined || layer === undefined) {
                return undefined;
            }

            if (!(layer.tagRenderings?.length > 0) || layer.title === undefined) {
                return undefined;
            }

            const tags = state.featureProperties.getStore(selectedElement.properties.id);
            return new SvelteUIElement(SelectedElementView, { state, layer, selectedElement, tags });
        },
        [selectedLayer]
    );

    const selectedElementTitle = selectedElement.map(
        (selectedElement) => {
            // Svelte doesn't properly reload some of the legacy UI-elements
            // As such, we _reconstruct_ the selectedElementView every time a new feature is selected
            // This is a bit wasteful, but until everything is a svelte-component, this should do the trick
            const layer = selectedLayer.data;
            if (selectedElement === undefined || layer === undefined) {
                return undefined;
            }

            const tags = state.featureProperties.getStore(selectedElement.properties.id);
            return new SvelteUIElement(SelectedElementTitle, { state, layer, selectedElement, tags });
        },
        [selectedLayer]
    );

    let mapproperties: MapProperties = state.mapProperties;
    let featureSwitches: FeatureSwitchState = state.featureSwitches;
    let availableLayers = state.availableLayers;
    let userdetails = state.osmConnection.userDetails;
    let currentViewLayer = layout.layers.find((l) => l.id === "current_view");
    let rasterLayer: Store<RasterLayerPolygon> = state.mapProperties.rasterLayer;
    let rasterLayerName =
        rasterLayer.data?.properties?.name ?? AvailableRasterLayers.maptilerDefaultLayer.properties.name;
    onDestroy(
        rasterLayer.addCallbackAndRunD((l) => {
            rasterLayerName = l.properties.name;
        })
    );
</script>

<div class="absolute top-0 left-0 h-screen w-screen overflow-hidden">
  <MaplibreMap map={maplibremap} />
</div>

<div class="pointer-events-none absolute top-0 left-0 w-full">
  <!-- Top components -->
  <If condition={state.featureSwitches.featureSwitchSearch}>
    <div class="pointer-events-auto float-right mt-1 px-1 max-[480px]:w-full sm:m-2">
      <Geosearch
        bounds={state.mapProperties.bounds}
        perLayer={state.perLayer}
        {selectedElement}
        {selectedLayer}
      />
    </div>
  </If>
  <div class="float-left m-1 flex flex-col sm:mt-2">
    <MapControlButton on:click={() => state.guistate.themeIsOpened.setData(true)}>
      <div class="m-0.5 mx-1 flex cursor-pointer items-center max-[480px]:w-full sm:mx-1 md:mx-2">
        <img class="mr-0.5 block h-6 w-6 sm:mr-1 md:mr-2 md:h-8 md:w-8" src={layout.icon} />
        <b class="mr-1">
          <Tr t={layout.title} />
        </b>
      </div>
    </MapControlButton>
    <MapControlButton on:click={() => state.guistate.menuIsOpened.setData(true)}>
      <MenuIcon class="h-8 w-8 cursor-pointer" />
    </MapControlButton>
    {#if currentViewLayer?.tagRenderings && currentViewLayer.defaultIcon()}
      <MapControlButton
        on:click={() => {
          selectedLayer.setData(currentViewLayer)
          selectedElement.setData(state.currentView.features?.data?.[0])
        }}
      >
        <ToSvelte
          construct={() => currentViewLayer.defaultIcon().SetClass("w-8 h-8 cursor-pointer")}
        />
      </MapControlButton>
    {/if}
    <ToSvelte
      construct={() => new ExtraLinkButton(state, layout.extraLink).SetClass("pointer-events-auto")}
    />
    <If condition={state.featureSwitchIsTesting}>
      <div class="alert w-fit">Testmode</div>
    </If>
  </div>
  <div class="flex w-full justify-center">
    <!-- Flex and w-full are needed for the positioning -->
    <!-- Centermessage -->
    <StateIndicator {state} />
  </div>
</div>

<div class="pointer-events-none absolute bottom-0 left-0 mb-4 w-screen">
  <!-- bottom controls -->
  <div class="flex w-full items-end justify-between px-4">
    <div class="flex flex-col">
      <If condition={featureSwitches.featureSwitchEnableLogin}>
        {#if state.lastClickObject.hasPresets || state.lastClickObject.hasNoteLayer}
          <button class="w-fit pointer-events-auto" on:click={() => {state.openNewDialog()}}>
            {#if state.lastClickObject.hasPresets}
              <Tr t={Translations.t.general.add.title} />
            {:else}
              <Tr t={Translations.t.notes.addAComment} />
            {/if}
          </button>
        {/if}
      </If>

      <div class="flex">
        <!-- bottom left elements -->
        <If condition={state.featureSwitches.featureSwitchFilter}>
          <MapControlButton on:click={() => state.guistate.openFilterView()}>
            <ToSvelte construct={Svg.filter_svg().SetClass("h-6 w-6")} />
          </MapControlButton>
        </If>
        <If condition={state.featureSwitches.featureSwitchBackgroundSelection}>
          <OpenBackgroundSelectorButton hideTooltip={true} {state} />
        </If>
        <a
          class="bg-black-transparent pointer-events-auto h-fit max-h-12 cursor-pointer self-end overflow-hidden rounded-2xl pl-1 pr-2 text-white opacity-50 hover:opacity-100"
          on:click={() => {
          state.guistate.themeViewTab.setData("copyright")
          state.guistate.themeIsOpened.setData(true)
        }}
        >
          © OpenStreetMap, <span class="w-24">{rasterLayerName}</span>
        </a>
      </div>
    </div>

    <div class="flex flex-col items-end">
      <!-- bottom right elements -->
      <If condition={state.floors.map((f) => f.length > 1)}>
        <div class="pointer-events-auto mr-0.5">
          <LevelSelector
            floors={state.floors}
            layerState={state.layerState}
            zoom={state.mapProperties.zoom}
          />
        </div>
      </If>
      <MapControlButton on:click={() => mapproperties.zoom.update((z) => z + 1)}>
        <ToSvelte construct={Svg.plus_svg().SetClass("w-8 h-8")} />
      </MapControlButton>
      <MapControlButton on:click={() => mapproperties.zoom.update((z) => z - 1)}>
        <ToSvelte construct={Svg.min_svg().SetClass("w-8 h-8")} />
      </MapControlButton>
      <If condition={featureSwitches.featureSwitchGeolocation}>
        <MapControlButton>
          <ToSvelte
            construct={new GeolocationControl(state.geolocation, mapproperties).SetClass(
              "block w-8 h-8"
            )}
          />
        </MapControlButton>
      </If>
    </div>
  </div>
</div>

<If
  condition={selectedElementView.map(
    (v) =>
      v !== undefined && selectedLayer.data !== undefined && !selectedLayer.data.popupInFloatover,
    [selectedLayer]
  )}
>
  <!-- right modal with the selected element view -->
  <ModalRight
    on:close={() => {
      selectedElement.setData(undefined)
    }}
  >
    <div class="normal-background absolute flex h-full w-full flex-col">
      <ToSvelte construct={new VariableUiElement(selectedElementTitle)}>
        <!-- Title -->
      </ToSvelte>
      <ToSvelte construct={new VariableUiElement(selectedElementView).SetClass("overflow-auto")}>
        <!-- Main view -->
      </ToSvelte>
    </div>
  </ModalRight>
</If>

<If
  condition={selectedElementView.map(
    (v) =>
      v !== undefined && selectedLayer.data !== undefined && selectedLayer.data.popupInFloatover,
    [selectedLayer]
  )}
>
  <!-- Floatover with the selected element, if applicable -->
  <FloatOver
    on:close={() => {
      selectedElement.setData(undefined)
    }}
  >
    <ToSvelte construct={new VariableUiElement(selectedElementView)} />
  </FloatOver>
</If>

<If condition={state.guistate.themeIsOpened}>
  <!-- Theme menu -->
  <FloatOver on:close={() => state.guistate.themeIsOpened.setData(false)}>
    <span slot="close-button"><!-- Disable the close button --></span>
    <TabbedGroup condition1={state.featureSwitches.featureSwitchFilter} tab={state.guistate.themeViewTabIndex}>
      <div slot="post-tablist">
        <XCircleIcon
          class="mr-2 h-8 w-8"
          on:click={() => state.guistate.themeIsOpened.setData(false)}
        />
      </div>

      <div class="flex" slot="title0">
        <img class="block h-4 w-4" src={layout.icon} />
        <Tr t={layout.title} />
      </div>

      <div class="m-4 h-full" slot="content0">
        <ThemeIntroPanel {state} />
      </div>

      <div class="flex" slot="title1">
        <ToSvelte construct={Svg.filter_svg().SetClass("w-4 h-4")} />
        <Tr t={Translations.t.general.menu.filter} />
      </div>

      <div class="m-2 flex flex-col" slot="content1">
        {#each layout.layers as layer}
          <Filterview
            zoomlevel={state.mapProperties.zoom}
            filteredLayer={state.layerState.filteredLayers.get(layer.id)}
            highlightedLayer={state.guistate.highlightedLayerInFilters}
          />
        {/each}
        {#each layout.tileLayerSources as tilesource}
          <OverlayToggle
            layerproperties={tilesource}
            state={state.overlayLayerStates.get(tilesource.id)}
            highlightedLayer={state.guistate.highlightedLayerInFilters}
            zoomlevel={state.mapProperties.zoom}
          />
        {/each}
      </div>

      <div class="flex" slot="title2">
        <If condition={state.featureSwitches.featureSwitchEnableExport}>
          <ToSvelte construct={Svg.download_svg().SetClass("w-4 h-4")} />
          <Tr t={Translations.t.general.download.title} />
        </If>
      </div>
      <div class="m-4" slot="content2">
        <DownloadPanel {state} />
      </div>

      <div slot="title3">
        <Tr t={Translations.t.general.attribution.title} />
      </div>

      <ToSvelte construct={() => new CopyrightPanel(state)} slot="content3" />

      <div class="flex" slot="title4">
        <ToSvelte construct={Svg.share_svg().SetClass("w-4 h-4")} />
        <Tr t={Translations.t.general.sharescreen.title} />
      </div>
      <div class="m-2" slot="content4">
        <ShareScreen {state} />
      </div>
    </TabbedGroup>
  </FloatOver>
</If>

<IfHidden condition={state.guistate.backgroundLayerSelectionIsOpened}>
  <!-- background layer selector -->
  <FloatOver on:close={() => {state.guistate.backgroundLayerSelectionIsOpened.setData(false)}}>
    <div class="h-full p-2">
      <RasterLayerOverview
        {availableLayers}
        map={state.map}
        mapproperties={state.mapProperties}
        userstate={state.userRelatedState}
        visible={state.guistate.backgroundLayerSelectionIsOpened}
      />
    </div>
  </FloatOver>
</IfHidden>

<If condition={state.guistate.menuIsOpened}>
  <!-- Menu page -->
  <FloatOver on:close={() =>      state.guistate.menuIsOpened.setData(false)    }>
    <span slot="close-button"><!-- Hide the default close button --></span>
    <TabbedGroup condition1={featureSwitches.featureSwitchEnableLogin} condition2={state.featureSwitches. featureSwitchCommunityIndex}
                 tab={state.guistate.menuViewTabIndex}>
      <div slot="post-tablist">
        <XCircleIcon
          class="mr-2 h-8 w-8"
          on:click={() => state.guistate.menuIsOpened.setData(false)}
        />
      </div>
      <div class="flex" slot="title0">
        <Tr t={Translations.t.general.menu.aboutMapComplete} />
      </div>

      <div class="links-as-button links-w-full m-2 flex flex-col gap-y-1" slot="content0">
        <Tr t={Translations.t.general.aboutMapComplete.intro} />

        <a class="flex" href={Utils.HomepageLink()}>
          <img class="h-6 w-6" src="./assets/svg/add.svg" />
          <Tr t={Translations.t.general.backToIndex} />
        </a>

        <a class="flex" href="https://github.com/pietervdvn/MapComplete/issues" target="_blank">
          <img class="h-6 w-6" src="./assets/svg/bug.svg" />
          <Tr t={Translations.t.general.attribution.openIssueTracker} />
        </a>

        <a class="flex" href="https://en.osm.town/@MapComplete" target="_blank">
          <img class="h-6 w-6" src="./assets/svg/mastodon.svg" />
          <Tr t={Translations.t.general.attribution.followOnMastodon} />
        </a>

        <a class="flex" href="https://liberapay.com/pietervdvn/" target="_blank">
          <img class="h-6 w-6" src="./assets/svg/liberapay.svg" />
          <Tr t={Translations.t.general.attribution.donate} />
        </a>

        <a class="flex" href={Utils.OsmChaLinkFor(7)} target="_blank">
          <img class="h-6 w-6" src="./assets/svg/statistics.svg" />
          <Tr t={Translations.t.general.attribution.openOsmcha.Subs({ theme: "MapComplete" })} />
        </a>
        {Constants.vNumber}
      </div>

      <div class="flex" slot="title1">
        <CogIcon class="h-6 w-6" />
        <Tr t={UserRelatedState.usersettingsConfig.title.GetRenderValue({})} />
      </div>

      <div class="links-as-button" slot="content1">
        <!-- All shown components are set by 'usersettings.json', which happily uses some special visualisations created specifically for it -->
        <LoginToggle {state}>
          <div class="flex flex-col" slot="not-logged-in">
            <ToSvelte construct={() => new LanguagePicker(layout.language, Locale.language)} />
            <Tr cls="alert" t={Translations.t.userinfo.notLoggedIn} />
            <LoginButton clss="primary" osmConnection={state.osmConnection} />
          </div>
          <SelectedElementView
            highlightedRendering={state.guistate.highlightedUserSetting}
            layer={UserRelatedState.usersettingsConfig}
            selectedElement={{
              type: "Feature",
              properties: {},
              geometry: { type: "Point", coordinates: [0, 0] },
            }}
            {state}
            tags={state.userRelatedState.preferencesAsTags}
          />
        </LoginToggle>
      </div>

      <div class="flex" slot="title2">
        <ToSvelte construct={Svg.community_svg().SetClass("w-6 h-6")} />
        <Tr t={Translations.t.communityIndex.title} />
      </div>
      <div class="m-2" slot="content2">
        <CommunityIndexView location={state.mapProperties.location} />
      </div>
      <div class="flex" slot="title3">
        <EyeIcon class="w-6" />
        <Tr t={Translations.t.privacy.title} />
      </div>
      <div class="m-2" slot="content3">
        <ToSvelte construct={() => new PrivacyPolicy()} />
      </div>

      <Tr slot="title4" t={Translations.t.advanced.title} />
      <div class="m-2 flex flex-col" slot="content4">
        <If condition={featureSwitches.featureSwitchEnableLogin}>
          <OpenIdEditor mapProperties={state.mapProperties} />
          <ToSvelte
            construct={() =>
            new OpenJosm(state.osmConnection, state.mapProperties.bounds).SetClass("w-full")}
          />
          <MapillaryLink mapProperties={state.mapProperties} />
        </If>

        <ToSvelte construct={Hotkeys.generateDocumentationDynamic} />
      </div>
    </TabbedGroup>
  </FloatOver>
</If>

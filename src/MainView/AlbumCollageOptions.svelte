<script>
  import { currentChartList, currentChartId } from "../store";
  import ChartOption from "../shared/ChartOption.svelte";
  import Button from "../shared/Button.svelte";

  let currentIndex = "";

  $: currentIndex = $currentChartList.findIndex(
    (chart) => chart.id === $currentChartId
  );

  const initialZoom = parseFloat(
    document.documentElement.style.getPropertyValue("--item-zoom")
  );

  let zoomValue = !initialZoom || initialZoom === NaN ? 1 : initialZoom;

  $: {
    document.documentElement.style.setProperty("--item-zoom", `${zoomValue}`);
  }

  //
  // Sorting the collage
  //

  // Sort functions we feed into the algorithm
  // Using title from image because it has both album and artist in a string
  const artistSort = (album1, album2) => {
    if (album1.title < album2.title) return -1;
    if (album1.title > album2.title) return 1;
    return 0;
  }

  // Using album
  const albumSort = (album1, album2) => {
    if (album1.album < album2.album) return -1;
    if (album1.album > album2.album) return 1;
    return 0;
  }

  // Main sorting function
  //
  // Known issue: sorting works with Album only if added from Chartr,
  // If an album is provided by a hyperlink, the hyperlink is set as 
  // the album title and it sorts by that rather than by the actual 
  // album name. There also may be album name glitching because of this.
  //
  const sortCollage = (sortMethod) => {

    // get collage list, flatten it, remove all undefined / nulls, sort it
    let listToSort = $currentChartList[currentIndex].albumCollageList;
    const rows = listToSort.length;
    const columns = listToSort[0].length;

    let listElements = listToSort.flat();
    listElements = listElements.filter(album => album != undefined);
    listElements.sort(sortMethod);

    // make new array with same dimensions as original list
    const rebuiltList = [];
    for (let row_index = 0; row_index < rows; row_index++) {
      let row = [];
      for (let column_index = 0; column_index < columns; column_index++) {
        let column = undefined;
        row.push(column);
      }
      rebuiltList.push(row);
    }

    // go through each row / column and place sorted values
    // in a spot with an already existing album art
    let listElementsIndex = 0;
    for (let row_index = 0; row_index < rows; row_index++) {
      for (let column_index = 0; column_index < columns; column_index++) {
        if (listToSort[row_index][column_index] != undefined) {
          rebuiltList[row_index][column_index] = listElements[listElementsIndex];
          listElementsIndex++;
        }
      }
    }

    // save to albumCollageList
    $currentChartList[currentIndex].albumCollageList = rebuiltList;
  };


</script>

<ChartOption
  type="text-range"
  label="Chart zoom:"
  labelFor="zoom"
  min={0.5}
  max={2}
  step={0.25}
  bind:value={zoomValue}
/>
<ChartOption
  type="checkbox"
  label="Show Album Titles"
  labelFor="show-album-titles"
  bind:value={$currentChartList[currentIndex].albumCollageOptions
    .showAlbumTitles}
/>
{#if $currentChartList[currentIndex].albumCollageOptions.showAlbumTitles}
  <ChartOption
    type="checkbox"
    label="Album Title Below Cover"
    labelFor="title-below-cover"
    bind:value={$currentChartList[currentIndex].albumCollageOptions
      .titlesBelowCover}
  />
  <ChartOption
    type="checkbox"
    label="Allow Editing Titles"
    labelFor="allow-edit-titles"
    bind:value={$currentChartList[currentIndex].albumCollageOptions
      .allowEditTitles}
  />
{/if}
<ChartOption
  type="range"
  label="Rows:"
  labelFor="rows"
  bind:value={$currentChartList[currentIndex].albumCollageOptions.rows}
/>
<ChartOption
  type="range"
  label="Columns:"
  labelFor="columns"
  bind:value={$currentChartList[currentIndex].albumCollageOptions.columns}
/>
<ChartOption
  type="text"
  label="Font:"
  labelFor="font"
  bind:value={$currentChartList[currentIndex].albumCollageOptions.font}
/>
<ChartOption
  type="text"
  label="Background Color / URL:"
  labelFor="bg-color"
  bind:value={$currentChartList[currentIndex].albumCollageOptions.background}
/>
<ChartOption
  type="text"
  label="Font Color:"
  labelFor="font-color"
  bind:value={$currentChartList[currentIndex].albumCollageOptions.fontColor}
/>
<ChartOption
  type="range"
  label="Padding:"
  labelFor="padding"
  bind:value={$currentChartList[currentIndex].albumCollageOptions.padding}
/>
<ChartOption
  type="range"
  label="Gap Between Items:"
  labelFor="gap"
  min="0"
  bind:value={$currentChartList[currentIndex].albumCollageOptions.gap}
/>

<div style={{width: "auto", margin: "auto"}}>
  <Button
    label="Sort by Album"
    id='sortAlbum'
    onClick={() => sortCollage(albumSort)}
  />
  <br />
  <Button
    label="Sort by Artist"
    id='sortArtist'
    onClick={() => sortCollage(artistSort)}
  />
</div>
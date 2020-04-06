<script>
  import Note from './Note.svelte';
  import { db } from './firebase';
  import { collectionData } from 'rxfire/firestore';
  import { startWith } from 'rxjs/operators';

  export let board;
  export let zoom = 1;

  $: if (zoom !== null) {
    if (document.querySelector('.Notes__scale')) {
      document.querySelector('.Notes__scale').style.transform = 'scale(' + zoom + ')';
    }
  }

  const query = db.collection('notes').where('board', '==', board);
  const notes = collectionData(query, 'id').pipe(startWith([]));

  function add() {
    if (document.querySelector('.Notes__translate').style.transform) {
      const translates = document.querySelector('.Notes__translate').style.transform.match(/[+-]?\d+(?:\.\d+)?/g).map(Number);

      originX = translates[0];
      originY = translates[1];
    } else {
      originX = 0;
      originY = 0;
    }

    const el = document.querySelector('.Notes__buttons');
    const x = (window.innerWidth / 2) + (((window.innerWidth / 2) - 20) / zoom) - 130 - originX;
    const y = (window.innerHeight / 2) - (((window.innerHeight / 2) - 50) / zoom) - originY;

    const text = '';
    db.collection('notes').add({ board: board, text: text, x: x, y: y, color: '#ffff80' });
  }

  function update(e) {
    const { id, text, x, y, color } = e.detail;

    db.collection('notes').doc(id).update({
      text: text,
      x: x,
      y: y,
      color: color
    });
  }

  function remove(e) {
    const { id } = e.detail;
    db.collection('notes').doc(id).delete();
  }

  let translateBoard = false;
  let mouseX, mouseY, originX, originY;

  function mousedown(e) {
    translateBoard = true;
    mouseX = e.clientX;
    mouseY = e.clientY;

    if (document.querySelector('.Notes__translate').style.transform) {
      const translates = document.querySelector('.Notes__translate').style.transform.match(/[+-]?\d+(?:\.\d+)?/g).map(Number);

      originX = translates[0];
      originY = translates[1];
    } else {
      originX = 0;
      originY = 0;
    }
  }

  function mousemove(e) {
    if (translateBoard) {
      const translateX = -(mouseX - e.clientX) / zoom + originX;
      const translateY = -(mouseY - e.clientY) / zoom + originY;

      document.querySelector('.Notes__translate').style.transform = 'translate(' + translateX + 'px, ' + translateY + 'px)';
    }
  }

  function mouseup() {
    translateBoard = false;
  }

  function resetZoom() {
    zoom = 1;
    document.querySelector('.Notes__translate').style.transform = '';
  }
</script>


<div class="Translator" on:mousedown={mousedown} on:mousemove={mousemove} on:mouseup={mouseup}></div>
<div class="Notes">
  <div class="Notes__scale">
    <div class="Notes__translate">
      {#each $notes as note}
        <Note {...note} on:update={update} on:remove={remove} />
      {/each}
    </div>
  </div>

  <div class="Notes__buttons">
    <div class="Notes__button" on:click={add}>➕</div> 
  </div>

  <div class="Notes__zoom">
    <input class="Notes__zoom-range" type="range" min="0.2" max="1.8" step="0.01" bind:value={zoom}>
    <div class="Notes__zoom-reset" on:click={resetZoom}>⚖️</div> 
  </div>
</div>

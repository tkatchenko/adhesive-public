<script>
  import { createEventDispatcher } from 'svelte';
  import { debounce } from './util.js';

  export let id;
  export let x;
  export let y;
  export let text;
  export let color;

  const dispatch = createEventDispatcher();

  function update() {
    dispatch('update', { id, text, x, y, color });
  }

  const deUpdate = debounce(() => update(), 500);

  function remove() {
    dispatch('remove', { id });
  }

  function keydown(e) {
    if (e.key === 'Enter' && !e.shiftKey) {
      e.preventDefault(); 
      e.target.blur();
    }

    deUpdate();
  }

  let mouseX, mouseY, mouseDragX, mouseDragY;

  function dragstart(e) {
    e.target.style.cursor = 'grabbing';

    for (let item of document.querySelectorAll('.Note')) {
      item.classList.remove('top');
    }
    e.target.classList.add('top');

    mouseX = e.offsetX;
    mouseY = e.offsetY;
  }

  function drag(e) {
    mouseDragX = e.offsetX;
    mouseDragY = e.offsetY;
  }

  function dragend(e) {
    x = (isNaN(x)) ? 0 : x;
    y = (isNaN(y)) ? 0 : y;

    x = parseInt(x, 10) + mouseDragX - mouseX;
    y = parseInt(y, 10) + mouseDragY - mouseY;

    e.target.style.cursor = 'grab';
    e.target.classList.remove('hide');

    update();
	}

  function changeColor() {
    const colors = [
      '#ffff80',
      '#ff8080',
      '#80ffff',
      '#ff80ff',
    ];

    const index = colors.indexOf(color);

    if (index >= 0) {
      color = colors[(index + 1) % colors.length];
    } else {
      color = colors[0];
    }

    update();
  }
</script>

<div
  class="Note"
  style="top: {y}px; left: {x}px; background-color: {color};"
  draggable="true"
  on:dragstart={dragstart}
  on:drag={drag}
  on:dragend={dragend}
>
  <div class="Note__text" contenteditable="plaintext-only" on:keydown={keydown} bind:innerHTML={text}></div>
  <div class="Note__buttons">
    <div class="Note__button Note__button--extra-padding" on:click={changeColor}>🌈 </div>
    <div class="Note__button" on:click={remove}>🗑 </div>
  </div>
</div>

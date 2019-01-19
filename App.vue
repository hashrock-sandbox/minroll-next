<template>
  <div>
    <svg width="600" height="600" viewBox="0 0 600 600">
      <rect
        v-for="(note, index) in dispNotes"
        :key="index"
        :x="note.x"
        :y="note.y"
        :width="note.width"
        :height="note.height"
      ></rect>
    </svg>
  </div>
</template>
<script lang="ts">
import { Transform, Rect } from "paintvec";
import Vue from "vue";

interface Note {
  noteNo: number;
  velocity: number;
  length: number;
  position: number;
}
interface NoteRect {
  x: number;
  y: number;
  height: number;
  width: number;
}

export default Vue.extend({
  data() {
    return {
      notes: [
        {
          noteNo: 0,
          velocity: 100,
          length: 480 * 16,
          position: 0
        },
        {
          noteNo: 127,
          velocity: 100,
          length: 480 * 16,
          position: 0
        }
      ] as Note[]
    };
  },
  computed: {
    dispNotes(): NoteRect[] {
      return this.notes.map(
        (i: Note): NoteRect => {
          let rect = Rect.fromWidthHeight(i.position, i.noteNo, i.length, 1);
          const range = 128;
          let from = Rect.fromWidthHeight(0, range, 480 * 32, -range);
          let to = Rect.fromWidthHeight(0, 0, 600, 600);

          let t = rect.transform(Transform.rectToRect(from, to));
          return {
            x: t.left,
            y: t.top,
            width: t.width,
            height: t.height
          };
        }
      );
    }
  }
});
</script>

<style>
svg {
  background: white;
}
body {
  background: #333;
}
</style>

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
<script>
import { Transform, Rect } from "paintvec";

export default {
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
      ]
    };
  },
  computed: {
    dispNotes() {
      return this.notes.map(i => {
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
      });
    }
  }
};
</script>

<style>
svg {
  background: white;
}
body {
  background: #333;
}
</style>

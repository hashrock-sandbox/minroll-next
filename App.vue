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
    <div style="background: #DDD;">
      <div>
        Start:
        <input type="range" min="0" max="10000" v-model.number="viewport.time.start">
      </div>
      <div>
        End:
        <input type="range" min="0" max="10000" v-model.number="viewport.time.end">
      </div>
      <div>
        NoteStart:
        <input type="range" min="0" max="128" v-model.number="viewport.note.start">
      </div>
      <div>
        NoteEnd:
        <input type="range" min="0" max="128" v-model.number="viewport.note.end">
      </div>
    </div>
  </div>
</template>
<script lang="ts">
import { Transform, Rect, Vec2 } from "paintvec";
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
const TIMEBASE = 480;
export default Vue.extend({
  data() {
    return {
      viewport: {
        note: {
          start: 12 * 2,
          end: 12 * 6
        },
        time: {
          start: 0,
          end: (TIMEBASE / 4) * 16 * 4
        }
      },
      notes: [
        {
          noteNo: 12 * 3,
          velocity: 100,
          length: (TIMEBASE / 4) * 1,
          position: 0
        },
        {
          noteNo: 12 * 4,
          velocity: 100,
          length: (TIMEBASE / 4) * 2,
          position: (TIMEBASE / 4) * 1
        }
      ] as Note[]
    };
  },
  computed: {
    dispNotes(): NoteRect[] {
      return this.notes.map(
        (i: Note): NoteRect => {
          let rect = Rect.fromWidthHeight(i.position, i.noteNo, i.length, 1);
          const noteRange = this.viewport.note.end - this.viewport.note.start;
          const timeRange = this.viewport.time.end - this.viewport.time.start;

          // let from = Rect.fromTwoPoints(
          //   new Vec2(this.viewport.time.start, this.viewport.note.end),
          //   new Vec2(this.viewport.time.end, this.viewport.note.start)
          // );
          let from = Rect.fromWidthHeight(
            this.viewport.time.start,
            this.viewport.note.end,
            timeRange,
            -noteRange
          );
          let to = Rect.fromWidthHeight(0, 0, 600, 600);

          let t = rect.transform(Transform.rectToRect(from, to));
          console.log(t);
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

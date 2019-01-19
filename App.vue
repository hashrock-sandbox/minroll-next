<template>
  <div>
    <svg width="600" height="600" viewBox="0 0 600 600" @mousemove="onMouseMove">
      <rect
        v-for="(note, index) in dispNotes"
        :key="index"
        :x="note.x"
        :y="note.y"
        :width="note.width"
        :height="note.height"
      ></rect>
      <rect
        fill="rgba(0, 0, 0, 0.3)"
        :x="dispPreview.x"
        :y="dispPreview.y"
        :width="dispPreview.width"
        :height="dispPreview.height"
        @click="addNote"
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

function toNoteRect(viewportTransform: Transform) {
  return function(i: Note): NoteRect {
    const rect = Rect.fromWidthHeight(i.position, i.noteNo, i.length, 1);
    const transformed = rect.transform(viewportTransform);
    return {
      x: transformed.left,
      y: transformed.top,
      width: transformed.width,
      height: transformed.height
    };
  };
}

function grid(size: number) {
  return function(x: number) {
    return Math.floor(x / size) * size;
  };
}

export default Vue.extend({
  data() {
    return {
      preview: {
        noteNo: 0,
        velocity: 100,
        length: 0,
        position: 0
      } as Note,
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
          noteNo: 12 * 4 + 1,
          velocity: 100,
          length: (TIMEBASE / 4) * 2,
          position: (TIMEBASE / 4) * 3
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
  methods: {
    onMouseMove(ev: MouseEvent) {
      const vec = new Vec2(ev.offsetX, ev.offsetY).transform(
        this.viewportTransform.invert()
      );
      this.preview = {
        position: grid(TIMEBASE / 4)(Math.floor(vec.x)),
        noteNo: Math.floor(vec.y),
        length: TIMEBASE / 4,
        velocity: 100
      };
    },
    addNote() {
      this.notes.push(this.preview);
    }
  },
  computed: {
    viewportTransform(): Transform {
      const noteRange = this.viewport.note.end - this.viewport.note.start;
      const timeRange = this.viewport.time.end - this.viewport.time.start;
      const from = Rect.fromWidthHeight(
        this.viewport.time.start,
        this.viewport.note.end,
        timeRange,
        -noteRange
      );
      const to = Rect.fromWidthHeight(0, 0, 600, 600);
      return Transform.rectToRect(from, to);
    },
    dispNotes(): NoteRect[] {
      return this.notes.map(toNoteRect(this.viewportTransform));
    },
    dispPreview(): NoteRect {
      return toNoteRect(this.viewportTransform)(this.preview);
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

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

      <rect
        :x="600 - 10"
        :y="dispViewport.y"
        :width="10"
        :height="dispViewport.height"
        @pointerdown="vScrollDown"
        @pointerup="vScroll = false"
        @pointermove="vScrollMove"
      ></rect>
      <rect
        :x="dispViewport.x"
        :y="0"
        :width="dispViewport.width"
        :height="10"
        @pointerdown="hScrollDown"
        @pointerup="hScroll = false"
        @pointermove="hScrollMove"
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
      track: {
        length: TIMEBASE * 4 * 4
      },
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
          end: (TIMEBASE / 4) * 16 * 2
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
      ] as Note[],
      hScroll: false,
      hScrollOffset: 0,
      vScroll: false,
      vScrollOffset: 0
    };
  },
  methods: {
    hScrollDown(e: PointerEvent) {
      const rect: any = e.target;
      rect.setPointerCapture(e.pointerId);
      const bbox = rect.getBoundingClientRect();
      this.hScrollOffset = e.clientX - bbox.left;
      this.hScroll = true;
    },
    hScrollMove(e: PointerEvent) {
      const rect: any = e.target;
      const bbox = rect.getBoundingClientRect();
      const x = e.clientX - bbox.left;
      if (this.hScroll) {
        let xpos = x - this.hScrollOffset;
        let start = new Vec2(xpos, 0).transform(this.viewportTransform.invert())
          .x;
        let range = this.viewport.time.end - this.viewport.time.start;
        this.viewport.time.start = start;

        //範囲制限
        if (this.viewport.time.start < 0) {
          this.viewport.time.start = 0;
        }
        if (this.viewport.time.start + range > this.track.length) {
          this.viewport.time.start = this.track.length - range;
        }
        this.viewport.time.end = this.viewport.time.start + range;
      }
    },
    vScrollDown(e: PointerEvent) {
      const rect: any = e.target;
      rect.setPointerCapture(e.pointerId);
      const bbox = rect.getBoundingClientRect();
      this.vScrollOffset = bbox.bottom - e.clientY;
      this.vScroll = true;
    },
    vScrollMove(e: PointerEvent) {
      const rect: any = e.target;
      const bbox = rect.getBoundingClientRect();
      const y = e.clientY + this.vScrollOffset;
      if (this.vScroll) {
        let range = this.viewport.note.end - this.viewport.note.start;
        this.viewport.note.start = ((600 - y) / 600) * 128;

        //範囲制限
        if (this.viewport.note.start < 0) {
          this.viewport.note.start = 0;
        }
        if (this.viewport.note.start + range > 128) {
          this.viewport.note.start = 128 - range;
        }
        this.viewport.note.end = this.viewport.note.start + range;
      }
    },

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
    },
    dispViewport() {
      const height = (this.viewport.note.end - this.viewport.note.start) / 128;

      return {
        x: (this.viewport.time.start / this.track.length) * 600,
        y: ((128 - this.viewport.note.start) / 128 - height) * 600,
        width:
          ((this.viewport.time.end - this.viewport.time.start) /
            this.track.length) *
          600,
        height: height * 600
      };
    }
  },
  mounted() {
    document.body.addEventListener("keydown", (ev: KeyboardEvent) => {
      if (ev.key === "ArrowLeft") {
        this.viewport.time.start -= TIMEBASE * 4;
        this.viewport.time.end -= TIMEBASE * 4;
      }
      if (ev.key === "ArrowRight") {
        this.viewport.time.start += TIMEBASE * 4;
        this.viewport.time.end += TIMEBASE * 4;
      }
      if (ev.key === "ArrowUp") {
        this.viewport.note.start += 4;
        this.viewport.note.end += 4;
      }
      if (ev.key === "ArrowDown") {
        this.viewport.note.start -= 4;
        this.viewport.note.end -= 4;
      }
    });
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

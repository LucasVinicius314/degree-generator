<template>
  <v-container class="fill-height">
    <v-responsive class="d-flex align-center text-center fill-height">
    </v-responsive>
  </v-container>
</template>

<script setup lang="ts">
import { onMounted } from "vue";

const frequencyFromNote = (note: number) => {
  return Math.pow(2, (note - 69) / 12) * 440;
};

onMounted(async () => {
  const midiAccess = await navigator.requestMIDIAccess();

  const availableMidiDevices = Array.from(midiAccess.inputs.values());

  console.log(availableMidiDevices);

  const context = new AudioContext();

  const oscillators: { [type: number]: OscillatorNode } = {};

  const playNote = (frequency: number) => {
    if (oscillators[frequency] === undefined) {
      oscillators[frequency] = context.createOscillator();
      oscillators[frequency].frequency.value = frequency;
    }

    oscillators[frequency].connect(context.destination);
    oscillators[frequency].start(context.currentTime);
  };

  const stopNote = (frequency: number) => {
    oscillators[frequency].stop(context.currentTime);
    oscillators[frequency].disconnect();
  };

  for (const midiDevice of availableMidiDevices) {
    midiDevice.addEventListener("midimessage", (e) => {
      const frequency = frequencyFromNote(e.data[1]);

      if (e.data[0] === 144 && e.data[2] > 0) {
        playNote(frequency);
      } else if (e.data[0] === 128 || e.data[2] === 0) {
        stopNote(frequency);
      }
    });
  }
});
</script>

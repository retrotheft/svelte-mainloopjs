<script lang="ts">
import { getContext, onDestroy } from 'svelte'
const { register, unregister, getFrame, getTimestamp } = getContext('loop');

export let name: string;
const dob = getTimestamp();
let age = 0;

function update(delta) {
  age += delta / 1000;
}

register('update', update)

onDestroy(() => unregister('update', update))
</script>

<div>{name} was born on {dob} has been <em>active</em> for {age.toFixed(0)}s</div>

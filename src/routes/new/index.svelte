<script lang="ts">
  import { goto } from '$app/navigation';
  import { loadingMeetingPromise, newMeeting } from '$lib/app-state';
  import { addMeeting } from '$lib/gql/addMeeting';
  import { meetingInput } from './_state/meeting';
  import { selectedDates } from './_state/intervals';
  import Head from '$lib/components/Head.svelte';
  import MeetingInput from './_MeetingInput.svelte';
  import Buttons from './_Buttons.svelte';

  const submit = () => {
    if ($selectedDates.value.length === 0) {
      $selectedDates.error = 'Required';
      return;
    }
    $loadingMeetingPromise = addMeeting($meetingInput);
    $newMeeting = { ...$meetingInput, slug: '' };
    goto('/loading');
  };
</script>

<Head emoji="✏️" subtitle="new event" />

<form on:submit|preventDefault={submit} class="max-w-lg p-6 mx-auto space-y-4">
  <MeetingInput />
  <Buttons />
</form>

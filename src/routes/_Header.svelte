<script lang="ts">
  import { receive, send } from '$lib/app-state';
</script>

<header
  in:receive={{ key: 'header' }}
  out:send={{ key: 'header' }}
  class="p-4 text-white card bg-gradient-primary bg-animate-slow"
>
  <h1 class="text-xl font-bold">meetwhen</h1>
  <p>Find the perfect time to meet up with your team</p>
</header>

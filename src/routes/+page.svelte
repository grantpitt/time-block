<script lang="ts">
  import { onMount } from "svelte";

  let notificationPermission: NotificationPermission | undefined = undefined;

  const durations = [25, 5, 25, 5, 25, 15] as const;
  $: uniqueDurations = [...new Set(durations)];
  let durationIndex = 0;
  $: duration = durations[durationIndex];

  $: timeRemaining = duration * 60;
  $: minutes = Math.floor(timeRemaining / 60);
  $: seconds = timeRemaining % 60;
  $: formatedTime = `${minutes}:${seconds < 10 ? "0" : ""}${seconds}`;
  let isTicking = false;
  let interval: number;

  onMount(async () => {
    notificationPermission = Notification.permission;
    if (notificationPermission === "default") {
      notificationPermission = await Notification.requestPermission();
    }
  });

  function toggle() {
    if (isTicking) {
      clearInterval(interval);
    } else {
      interval = setInterval(() => {
        timeRemaining -= 1;
        if (timeRemaining <= 0) {
          console.log("calling done in toggle interval");
          isTicking = false;
          clearInterval(interval);
          done();
        }
      }, 1000);
    }

    console.log("calling toggle");
    isTicking = !isTicking;
  }

  function done() {
    durationIndex = (durationIndex + 1) % durations.length;
    if (notificationPermission === "granted") {
      new Notification("Time Block", {
        body: "Timer is done",
      });
    }
  }

  function handleDurrationClick(duration: (typeof durations)[number]) {
    durationIndex = durations.indexOf(duration);
  }
</script>

<svelte:head>
  <title>{formatedTime}</title>
</svelte:head>

<main>
  <h1>Time Block</h1>
  <p>
    It might be interesting to see if time blocking helps in studying - some promonent programmers
    that describe the "ram loading" required to get into deep work might initially suggest this
    [time blocking] is a terrible self-sabatoging act. However, maybe the break interval can be less
    of an interuption and more of a cool-down where you can still think about the problems. The
    concept of having an insight into a problem in the shower comes to mind.
  </p>

  <article>
    {#each uniqueDurations as dur}
      <button on:click={() => handleDurrationClick(dur)}>
        {dur}
      </button>
    {/each}
  </article>

  <code>{formatedTime}</code>

  <button on:click={toggle}>
    {isTicking ? "Stop" : "Start"}
  </button>
</main>

<style global>
  @import url("https://fonts.googleapis.com/css2?family=Martian+Mono&display=swap");

  :global(body),
  :global(html) {
    font-family: sans-serif;
    padding: 0;
    margin: 0;
    background-color: #857492;
  }

  main {
    max-width: 680px;
    margin: 0 auto;
    padding: 1rem;
    background-color: #c2ff7d;
    border: 2px dashed #000000;
    border-top: none;
  }

  article {
    display: flex;
    gap: 1rem;
  }

  code {
    font-size: 5rem;
    font-family: "Martian Mono", monospace;
    display: block;
    text-align: center;
  }
</style>

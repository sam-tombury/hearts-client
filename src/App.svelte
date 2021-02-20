<script>
  import router from "page";

  import Lobby from "./routes/Lobby.svelte";
  import Game from "./routes/Game.svelte";

  let page;
  let params;

  router("/", () => (page = Lobby));
  router("/play/game/:gameID/seat/:seatID", (ctx) => {
    page = Game;
    params = ctx.params;
  });

  router.start();
</script>

<main>
  <svelte:component this={page} {params} />
</main>

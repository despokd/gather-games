<script>
  import data from "./games.json";
  import "carbon-components-svelte/css/all.css";
  import LogoGithub20 from "carbon-icons-svelte/lib/LogoGithub20";
  import Information20 from "carbon-icons-svelte/lib/Information20";
  import Play20 from "carbon-icons-svelte/lib/Play20";
  import Add20 from "carbon-icons-svelte/lib/Add20";
  import Close20 from "carbon-icons-svelte/lib/Close20";
  import Renew20 from "carbon-icons-svelte/lib/Renew20";
  import Copy20 from "carbon-icons-svelte/lib/Copy20";
  import Launch20 from "carbon-icons-svelte/lib/Launch20";
  import User16 from "carbon-icons-svelte/lib/User16";
  import Star16 from "carbon-icons-svelte/lib/Star16";
  import {
    Button,
    Form,
    Header,
    HeaderUtilities,
    HeaderActionLink,
    HeaderSearch,
    Loading,
    OutboundLink,
    SkipToContent,
    TextInput,
    ClickableTile,
    Content,
    Grid,
    Row,
    Column,
  } from "carbon-components-svelte";
  import copy from "clipboard-copy";

  /** @type {"white" | "g10" | "g90" | "g100"} */
  let theme;
  $: document.documentElement.setAttribute("theme", theme);
  /**
   * Sets theme color based on user preferences
   */
  function setTheme() {
    theme = window.matchMedia("(prefers-color-scheme: dark)").matches
      ? "g100"
      : "white";
  }
  // set theme on mode change
  window
    .matchMedia("(prefers-color-scheme: dark)")
    .addEventListener("change", () => {
      setTheme();
    });

  // init theme
  setTheme();

  /** order games by name */
  const orderBy = (arr, props, orders = "asc") =>
    [...arr].sort((a, b) =>
      props.reduce((acc, prop, i) => {
        if (acc === 0) {
          const [p1, p2] =
            orders && orders[i] === "desc"
              ? [b[prop], a[prop]]
              : [a[prop], b[prop]];
          acc = p1 > p2 ? 1 : p1 < p2 ? -1 : 0;
        }
        return acc;
      }, 0)
    );
  let games = orderBy(data.games, ["name"], "desc");

  /** join/game url  */
  let joinLink = "";
  let showIframe = false;
  let iframeUrl = "";

  $: iframeUrl = joinLink && joinLink != "" ? joinLink : "";
  $: showIframe = iframeUrl && iframeUrl != "";

  function scrollTo(id) {
    document.getElementById(id).scrollIntoView({
      behavior: "smooth",
    });
  }

  /** show iframe loading new src */
  let iframeLoading = false;
  $: if (iframeUrl) {
    iframeLoading = true;
  }

  /** instant open games by URL param */
  let getGameId = new URLSearchParams(window.location.search).get("g");
  let getGame = games.find((o) => o.id === getGameId);

  $: joinLink = getGame ? getGame.url : "";

  /** Search */
  let s_data = data.games;
  let s_value = "";
  let s_selectedResultIndex = 0;

  /* format data for search results */
  const renameKeys = (keysMap, obj) =>
    Object.keys(obj).reduce(
      (acc, key) => ({
        ...acc,
        ...{ [keysMap[key] || key]: obj[key] },
      }),
      {}
    );

  // format item for search results
  Object.entries(s_data).forEach((item) => {
    item[1] = renameKeys({ name: "text", desc: "description" }, item[1]);
    s_data[item[0]] = item[1];
  });

  $: lowerCaseValue = s_value.toLowerCase();
  $: s_results =
    s_value.length > 0
      ? s_data.filter((item) => {
          return (
            item.text.toLowerCase().includes(lowerCaseValue) ||
            item.description.includes(lowerCaseValue)
          );
        })
      : [];
</script>

<Header company="Techillian" platformName="Games">
  <div slot="skip-to-content">
    <SkipToContent />
  </div>

  <HeaderUtilities>
    <HeaderSearch
      bind:value={s_value}
      bind:selectedResultIndex={s_selectedResultIndex}
      placeholder="Search games"
      bind:results={s_results}
      on:select={(e) => {
        iframeUrl = e.detail.selectedResult.url;
      }}
    />
    <HeaderActionLink
      aria-label="View project on Github"
      icon={LogoGithub20}
      href="https://github.com/Techillian/gather-games"
      target="_blank"
    />
    <HeaderActionLink
      aria-label="About Techillian"
      icon={Information20}
      href="https://techillian.com"
      target="_blank"
    />
  </HeaderUtilities>
</Header>

<Content>
  <Grid>
    <Row>
      <Column>
        <h1>Games for gather.town</h1>
        <p>
          Create a new game session and share the link to your friends or enter
          the given link to play in gather.town
        </p>
        <Form
          class="inline-form"
          on:submit={() => {
            iframeUrl = joinLink;
            scrollTo("anchorEmbedGame");
          }}
        >
          <TextInput
            name="join"
            hideLabel
            labelText="Join game"
            placeholder="Enter link to hosted game"
            bind:value={joinLink}
          />
          <Button type="submit" size="field" icon={Add20}>Join</Button>
        </Form>
      </Column>
    </Row>
  </Grid>

  <span id="anchorEmbedGame" />
  {#if showIframe}
    <Row class="embedGame-row">
      <Column style="padding: 0;">
        <Button
          icon={Close20}
          kind="danger"
          size="small"
          on:click={() => {
            showIframe = false;
            iframeUrl = "";
          }}>Close</Button
        >
        <Button
          icon={Renew20}
          kind="secondary"
          size="small"
          on:click={() => (iframeUrl += " ")}>Refresh</Button
        >
        <Button
          icon={Copy20}
          kind="secondary"
          size="small"
          on:click={() => copy(iframeUrl)}>Get link</Button
        >
        <div class="iframe-wrapper">
          {#if iframeLoading}
            <div class="embedGame loading-wrapper">
              <Loading withOverlay={false} />
            </div>
          {/if}
          <iframe
            id="embedGame"
            title="game"
            class="embedGame"
            class:embedGame-none={iframeLoading}
            src={iframeUrl}
            on:load={() => (iframeLoading = false)}
          />
        </div>
        <Button
          kind="secondary"
          icon={Launch20}
          size="small"
          href={iframeUrl}
          target="_blank">{iframeUrl}</Button
        >
      </Column>
    </Row>
  {/if}

  <Grid>
    <Row style="margin-top: var(--cds-spacing-06);">
      {#each games as game, i}
        <Column
          sm={4}
          md={8}
          lg={8}
          xlg={4}
          style="margin-bottom: var(--cds-spacing-06);"
        >
          <ClickableTile
            id={"g-" + game.id}
            class="game-tile{game.fav ? ' fav' : ''}"
            on:click={() => {
              iframeUrl = game.url;
              scrollTo("anchorEmbedGame");
            }}
          >
            <div
              class="game-wrapper"
              style="background-image: url('img/g-{game.id}.png');"
            >
              <div>
                <h2>{game.name}</h2>
                <p>{game.desc}</p>
              </div>
              <div>
                <div class="custom-tag-wrapper">
                  <div
                    class="custom-tag"
                    title="{game.players.min} to {game.players
                      .max} players possible"
                  >
                    <User16 />{game.players.min} - {game.players.max}
                  </div>
                  {#if game.fav}
                    <div class="custom-tag" title="Recommended">
                      <Star16 />
                    </div>
                  {/if}
                </div>

                <Button
                  icon={Play20}
                  size="small"
                  on:click={() => {
                    iframeUrl = game.url;
                    scrollTo("anchorEmbedGame");
                  }}
                >
                  Play
                </Button>
              </div>
            </div>
          </ClickableTile>
        </Column>
      {/each}
    </Row>
  </Grid>
  <Grid>
    <Row>
      <Column>
        <footer>
          &copy; {new Date().getFullYear()} Techillian -
          <OutboundLink href="https://techillian.com/imprint"
            >Imprint</OutboundLink
          >
        </footer>
      </Column>
    </Row>
  </Grid>
</Content>

<style>
</style>

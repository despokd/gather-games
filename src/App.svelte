<script>
  import data from "./games.json";
  import LogoGithub20 from "carbon-icons-svelte/lib/LogoGithub20";
  import Information20 from "carbon-icons-svelte/lib/Information20";
  import Play20 from "carbon-icons-svelte/lib/Play20";
  import Add20 from "carbon-icons-svelte/lib/Add20";
  import User16 from "carbon-icons-svelte/lib/User16";
  import {
    Button,
    Form,
    Header,
    HeaderUtilities,
    HeaderActionLink,
    HeaderSearch,
    SkipToContent,
    TextInput,
    ClickableTile,
    Content,
    Grid,
    Row,
    Column,
  } from "carbon-components-svelte";

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

  /** join url */
  let joinLink = "";

  function openLink(url = null) {
    url = url == null ? joinLink : url;
    window.open(url, "_self");
  }

  /** order games by name */
  const orderBy = (arr, props, orders) =>
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

  $: console.log("value", s_value);
  $: console.log("result", s_results);
  $: console.log("selectedResultIndex", s_selectedResultIndex);
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
        openLink(e.detail.selectedResult.url);
      }}
    >
      <span slot="result"> x </span>
    </HeaderSearch>
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
        <Form class="inline-form" on:submit={() => openLink()}>
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

    <Row style="margin-top: var(--cds-spacing-06);">
      {#each games as game, i}
        <Column
          sm={4}
          md={8}
          lg={4}
          style="margin-bottom: var(--cds-spacing-06);"
        >
          <ClickableTile href={game.url} id={"g-" + game.id} class="game-tile">
            <div
              class="game-wrapper"
              style="background-image: url('img/g-{game.id}.png');"
            >
              <div>
                <h2>{game.name}</h2>
                <p>{game.desc}</p>
              </div>
              <div>
                <div class="custom-tag">
                  <User16 />{game.players.min} - {game.players.max}
                </div>

                <Button icon={Play20} size="small" href={game.url}>Play</Button>
              </div>
            </div>
          </ClickableTile>
        </Column>
      {/each}
    </Row>
  </Grid>
</Content>

<style>
</style>

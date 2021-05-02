<script>
  import data from "./games.json";
  import LogoGithub20 from "carbon-icons-svelte/lib/LogoGithub20";
  import Information20 from "carbon-icons-svelte/lib/Information20";
  import {
    Button,
    Form,
    Header,
    HeaderUtilities,
    HeaderActionLink,
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

  function openLink() {
    console.log(joinLink);
    window.open(joinLink, "_self");
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
</script>

<Header company="Techillian" platformName="Games">
  <div slot="skip-to-content">
    <SkipToContent />
  </div>

  <HeaderUtilities>
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
          <Button type="submit" size="field">Join game</Button>
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
          <ClickableTile
            id={"g-" + game.id}
            class="game-tile"
            style="background-image: url('img/g-{game.id}.png');"
          >
            <div>
              <h2>{game.name}</h2>
              <p>{game.desc}</p>
            </div>
            <div><Button size="small" href={game.url}>Play game</Button></div>
          </ClickableTile>
        </Column>
      {/each}
    </Row>
  </Grid>
</Content>

<style>
</style>

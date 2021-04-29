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
    Tile,
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

  let joinLink;
</script>

<Header company="Techillian" platformName="Games">
  <div slot="skip-to-content">
    <SkipToContent />
  </div>

  <Form class="inline-form" on:submit>
    <TextInput
      name="join"
      inline
      hideLabel
      labelText="Join game"
      placeholder="Enter link to hosted game"
      :value={joinLink}
    />
    <Button type="submit" size="field">Join game</Button>
  </Form>

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
      </Column>
    </Row>

    <Row style="margin-top: var(--cds-spacing-06);">
      {#each data.games as game, i}
        <Column>
          <Tile id={"g-" + game.id}>
            <h2>{game.name}</h2>
            <p>{game.desc}</p>
            <Button size="small" href={game.url}>Play game</Button>
          </Tile>
        </Column>
      {/each}
    </Row>
  </Grid>
</Content>

<style>
</style>

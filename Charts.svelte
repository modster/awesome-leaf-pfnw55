<script>
  // data from https://cdn.jsdelivr.net/gh/akabab/superhero-api@0.3.0/api/all.json
  import people from "./public/data.json";
  import { uniq } from "lodash";

  import Chart from "./Chart.svelte";
  const filteredPeople = people.filter(
    d => d.biography.publisher === "Marvel Comics"
  );

  const universes = [
    "Avengers",
    "Hulk",
    "Captain America",
    "Spider-man",
    "X-men",
    "Star Wars",
    "Batman",
    "Thor",
    "Deadpool",
    "Daredevil",
    "Fantastic Four",
    "Justice League",
    "Wonder Woman",
    "Superman",
    "Teenage Mutant Ninja Turtles"
  ];
  let universe = universes[0];
  $: universePeople = people.filter(
    d =>
      d.biography.firstAppearance
        .toLowerCase()
        .includes(universe.toLowerCase()) ||
      d.biography.publisher?.toLowerCase().includes(universe.toLowerCase()) ||
      d.connections.groupAffiliation
        ?.toLowerCase()
        .includes(universe.toLowerCase()) ||
      d.name.toLowerCase().includes(universe.toLowerCase())
  );

  const goodPeople = filteredPeople.filter(d => d.biography.alignment === "good");
  const badPeople = filteredPeople.filter(d => d.biography.alignment === "bad");

  const metrics = Object.keys(people[0].powerstats);
  let metric = metrics[0];
  let metric2 = metrics[1];
</script>

<select bind:value={universe} style="position: relative; padding: 0.3em; margin-bottom: 2em; z-index: 10">
  {#each universes as universe}
    <option value={universe}>{universe}</option>
  {/each}
</select>
<select bind:value={metric} style="position: relative; padding: 0.3em; margin-bottom: 2em; z-index: 10">
  {#each metrics as metric}
    <option>{metric}</option>
  {/each}
</select>
<select bind:value={metric2} style="position: relative; padding: 0.3em; margin-bottom: 2em; z-index: 10">
  {#each metrics as metric}
    <option>{metric}</option>
  {/each}
</select>
<!-- <Chart metric={metric} people={people} allPeople={people} /> -->
<div>
  <strong>Heroes</strong> ({universePeople.length})
  <Chart metric1={metric} metric2={metric2} people={universePeople} allPeople={people} />
<!-- 
  <br />

  <strong>Villans</strong>
  <Chart metric={metric} people={badPeople} allPeople={people} /> -->
</div>
---
tags: gamedev
excerpt-separator: <!--more-->
---
# Distilling videogame genres

People say that when you design a game, the first question you should think about is: what do I want
players to experience?

[I've been creating tiny games](https://jmmut.itch.io/) for some years now. When designing a new
game, even before choosing a game genre, recently I explicitly realised that I have to choose how
hard is it going to feel.

"Hard" in videogames, as far as I can see, can be either physically hard, or mentally hard.

<!--more-->

- Physically challenging games will require different button presses with specific timing. This
requires sensory-motor coordination, similar to playing a physical sport, or playing an instrument.

- On the other side, mentally challenging obviously means that you have to solve problems, plan,
predict, etc., or you won't succeed.

Different people get different levels of gratification when succeeding in a physical or mental
challenge. Sometimes people don't even want to face challenges and just want easy entertainment.

(Anecdotal tangent, it seems [academia is sometimes biased to favour *challenging* research, rather
than *useful* research](https://www.youtube.com/watch?v=6ED36HvQSvk). This video starts talking
about programming languages, but I promise it *is* about biases on the value of challenge.)

So I'm considering whether it's a good design strategy to think about how challenging I want the
game to be, and then choose a genre based on that.

## Classifying genres by challenge

These are the main videogame genres [according to
wikipedia](https://en.wikipedia.org/wiki/List_of_video_game_genres), and my rating on how
challenging they usually are:

<!--
| genre | physical challenge | mental challenge | example subgenres |
|--|----------|--------| -- |
|action| `*****` | `***` | platformers, shooters |
|role-playing | `***` | `***` | ARPG, MMORPG, roguelikes |
|sports | `***` | `***` | racing, (real) sports |
|action-adventure | `***` | `****` | metroidvanias, horror |
|simulation | `**` | `**` | life simulation, vehicle simulation. (racing?) |
|strategy | `**` | `*****` | 4X, Real-time Strategy, (MOBA?) |
|adventure | `*` | `****` | text adventures, visual novels, interactive movie |
|puzzle | `*` | `*****` | logic, exploration |
-->


<table id="myTable2">
  <thead>
    <tr>
      <th onclick="sortTable(0)">⇅ genre</th>
      <th onclick="sortTable(1)">⇅ physical challenge</th>
      <th onclick="sortTable(2)">⇅ mental challenge</th>
      <th>example subgenres</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>action</td>
      <td><code class="language-plaintext highlighter-rouge">*****</code></td>
      <td><code class="language-plaintext highlighter-rouge">***</code></td>
      <td>platformers, shooters</td>
    </tr>
    <tr>
      <td>role-playing</td>
      <td><code class="language-plaintext highlighter-rouge">***</code></td>
      <td><code class="language-plaintext highlighter-rouge">***</code></td>
      <td><span class="tooltip">ARPG<span class="tooltiptext">Action Role-Playing Game</span></span>,
 <span class="tooltip">MMORPG<span class="tooltiptext">Massively Multiplayer Online Role-Playing Game</span></span>, roguelikes</td>
    </tr>
    <tr>
      <td>sports</td>
      <td><code class="language-plaintext highlighter-rouge">***</code></td>
      <td><code class="language-plaintext highlighter-rouge">***</code></td>
      <td>racing, (real) sports</td>
    </tr>
    <tr>
      <td>action-adventure</td>
      <td><code class="language-plaintext highlighter-rouge">***</code></td>
      <td><code class="language-plaintext highlighter-rouge">****</code></td>
      <td>metroidvanias, horror</td>
    </tr>
    <tr>
      <td>simulation</td>
      <td><code class="language-plaintext highlighter-rouge">**</code></td>
      <td><code class="language-plaintext highlighter-rouge">**</code></td>
      <td>life simulation, vehicle simulation. (racing?)</td>
    </tr>
    <tr>
      <td>strategy</td>
      <td><code class="language-plaintext highlighter-rouge">**</code></td>
      <td><code class="language-plaintext highlighter-rouge">*****</code></td>
      <td><span class="tooltip">4X<span class="tooltiptext">Explore, Expand, Exploit,
Exterminate</span></span>, <span class="tooltip">RTS<span class="tooltiptext">Real-Time
Strategy</span></span>, (<span class="tooltip">MOBA<span class="tooltiptext">Multiplayer Online Battle Arena</span></span>?)</td>
    </tr>
    <tr>
      <td>adventure</td>
      <td><code class="language-plaintext highlighter-rouge">*</code></td>
      <td><code class="language-plaintext highlighter-rouge">****</code></td>
      <td>text adventures, visual novels, interactive movie</td>
    </tr>
    <tr>
      <td>puzzle</td>
      <td><code class="language-plaintext highlighter-rouge">*</code></td>
      <td><code class="language-plaintext highlighter-rouge">*****</code></td>
      <td>logic, exploration</td>
    </tr>
  </tbody>
</table>

Let me add some notes about that table:
- Any competitive multiplayer game will have the added mental challenge of game theory.
- Some subgenres fall within several main genres, e.g. point-and-click can be both adventure and
puzzle.
- Some genres like <span class="tooltip">RPG<span class="tooltiptext">Role-Playing
Game</span></span> have high variability how challenging they are, which can't be represented
with a single number. Don't yell at me, I wanted a simple overview.

## Choosing genre based on chosen difficulty

So, if I want to design a game that is mildly physically challenging, and very mentally challenging,
maybe it makes sense to make a strategy game? Or I could push the limits and make a physically
challenging puzzle, or a very complicated metroidvania?

Maybe this frame of mind is too limiting? I still like this kind of mental models, because it allows
you to look at the gaps:

- There seems to be more mental challenges than physical challenges. Why is this? Could I exploit
that? are computer-things intrinsically more mental than physical?
- Not many games are both physically and mentally challenging. Is this because people don't usually
enjoy *both* challenges at the same time? In the same sense that a game with many genres only appeal
to people which enjoy all those genres?

Another way to use this mental model is, I could try to subvert the genre expectations:
- Make a puzzle game that is easy to understand and solve but requires accurate timing and presses?
- A <span class="tooltip">4X<span class="tooltiptext">Explore, Expand, Exploit,
Exterminate</span></span>, that requires minimal thinking, for example framed
as a clicker game?

Or provide a structured understanding of why original games are original and/or successful:
- Flappy bird could be an action game where the physical difficulty is just timing when pressing a single
button, not coordination with other buttons.
- Cookie Clicker is neither physically nor mentally challenging (unless you try to maximize the
rewards), but is surreally funny and still manipulates our dopamine with "number go up".
- What kinds of mental challenges exist? Social games like Among Us have a bigger audience than
other strategy games because it leverages the idea (IMO) that most people enjoy social mental
challenges (game theory) more than abstract mental challenges (like numeric reward optmization).


## Limitations of the model

In terms of distilling the genres, of course the 2-dimensional classification of mental and physical
challenges is insufficient to generate all the genres, for 2 main reasons: it doesn't take into
account other important aspects like how important is the narrative (including the theme and
aesthetics), and the fact that 2 mechanics can be equally challenging but produce different
experiences.

After thinking about this for a bit, I would say that a way to understand game genres (ultimately
what kind of experiences a game produces) is to identify how much it has of:

- Physical difficulty
- Mental difficulty
- Narrative
- Aesthetics
- Social interaction


<script>
// https://www.w3schools.com/howto/howto_js_sort_table.asp
function sortTable(n) {
  var table, rows, switching, i, x, y, shouldSwitch, dir, switchcount = 0;
  table = document.getElementById("myTable2");
  switching = true;
  // Set the sorting direction to ascending:
  dir = "asc";
  /* Make a loop that will continue until
  no switching has been done: */
  while (switching) {
    // Start by saying: no switching is done:
    switching = false;
    rows = table.rows;
    /* Loop through all table rows (except the
    first, which contains table headers): */
    for (i = 1; i < (rows.length - 1); i++) {
      // Start by saying there should be no switching:
      shouldSwitch = false;
      /* Get the two elements you want to compare,
      one from current row and one from the next: */
      x = rows[i].getElementsByTagName("TD")[n];
      y = rows[i + 1].getElementsByTagName("TD")[n];
      /* Check if the two rows should switch place,
      based on the direction, asc or desc: */
      if (dir == "asc") {
        if (x.innerHTML.toLowerCase() > y.innerHTML.toLowerCase()) {
          // If so, mark as a switch and break the loop:
          shouldSwitch = true;
          break;
        }
      } else if (dir == "desc") {
        if (x.innerHTML.toLowerCase() < y.innerHTML.toLowerCase()) {
          // If so, mark as a switch and break the loop:
          shouldSwitch = true;
          break;
        }
      }
    }
    if (shouldSwitch) {
      /* If a switch has been marked, make the switch
      and mark that a switch has been done: */
      rows[i].parentNode.insertBefore(rows[i + 1], rows[i]);
      switching = true;
      // Each time a switch is done, increase this count by 1:
      switchcount ++;
    } else {
      /* If no switching has been done AND the direction is "asc",
      set the direction to "desc" and run the while loop again. */
      if (switchcount == 0 && dir == "asc") {
        dir = "desc";
        switching = true;
      }
    }
  }
}
</script>

<script>
  import Drop from "./Drop.svelte";

  let files = []; // array of: { filename, size, text }, only one file drop is allowed
  let input = ""; // textarea
  let output = ""; // textarea
  let feedRatesFound = []; // display at end of UI
  let feedModifier = 1.0; // adjusted by input range slider

  const handleFile = ({ detail: file }) => {
    // not used
  };

  const handleRead = ({ detail: { data, file } }) => {
    files.push({
      name: file.name,
      size: file.size,
      data,
    });

    files = files; // update state

    // update textarea text content
    input = data;

    modify(data);
  };

  const modify = (data) => {
    // parse each line of G-code data
    const lines = data.split(/\r?\n/);

    // loop through each line
    // look for lines with F* values, if found, modify the line, otherwise return line as-is

    // regex for floating point numbers
    // https://stackoverflow.com/questions/12643009/regular-expression-for-floating-point-numbers

    const newLines = lines.map((line) => {
      const f = line.match(/F([+-]?([0-9]*[.])?[0-9]+)/);

      if (f) {
        // if f does not exist in array feedRatesFound, add it
        if (!feedRatesFound.includes(f[1])) {
          feedRatesFound.push(f[1]);
          feedRatesFound = feedRatesFound; // force state update
        }

        const newF = f[1] * feedModifier;

        // replace with new modified feed rate, and remove final '.' if present
        return line
          .replace(/F([+-]?([0-9]*[.])?[0-9]+)/, `F${newF.toFixed(2)}`)
          .replace(/\.+$/, "");
      } else {
        return line;
      }
    });

    // will write to output textarea
    output = newLines.join("\n");
  };

  // download output as text file using uri encoding
  const download = () => {
    const element = document.createElement("a");
    const file = new Blob([output], { type: "text/plain" });
    element.href = URL.createObjectURL(file);

    // add "-modified" to file base name
    const filename = files[0].name.replace(/(\.[^/.]+)$/, "-modified$1");
    element.download = filename;

    document.body.appendChild(element); // Required for this to work in FireFox
    element.click();
  };
</script>

<h2>Feed rate modifier for G-code</h2>

<div class="input">
  <input
    type="range"
    min="0.10"
    max="4.00"
    step="0.02"
    bind:value={feedModifier}
    on:change={modify(input)}
  />
  <span>Feed modifier {Math.round(feedModifier * 100)}%</span>
</div>
<div>
  <button
    disabled={input === ""}
    on:click={() => {
      download();
    }}>Download output</button
  >
</div>

<div>
  <Drop
    readAsText
    on:drop={() => {
      // clear files on each drop event
      files = [];
      files = files;

      // clear saved feed rates
      feedRatesFound = [];
    }}
    on:file={handleFile}
    on:read={handleRead}
  >
    <textarea
      id="before"
      cols="80"
      rows="25"
      bind:value={input}
      placeholder="Drag and drop input file here"
    />
  </Drop>

  <textarea
    class={input === "" ? "hide" : ""}
    id="after"
    cols="80"
    rows="25"
    bind:value={output}
  />
</div>

<div>
  Following feedrates were found in input data: {feedRatesFound.join(" ")}
</div>

<footer>
  <a href="https://github.com/andygock/feed-modifier">GitHub</a>
</footer>

<style>
  textarea {
    font-family: monospace;
  }
  .input {
    display: flex;
    align-items: center;
  }
  .input input {
    margin-right: 0.5rem;
  }
  .hide {
    display: none;
  }
  footer {
    text-align: right;
    font-size: small;
  }
</style>

<script>
  import { tick } from "svelte";
  import { POPULAR_CITIES, STORAGE_KEY, LocalStorage, capitalCase } from './utils';

  const inputs = {
    cities: "",
    otherAlsoSearchFor: "",
    otherExcludedKeywords: '',
  };
  const checkboxes = {
    nearMe: false,
    verifiedOnly: false,
    excludeUnverified: false,
  };
  const alsoSearchFor = {
    ICU: {
      keywords: ["icu"],
      checked: false,
    },
    ventilator: {
      keywords: ["ventilator"],
      checked: false
    },
    oxygen: {
      keywords: ["oxygen"],
      checked: false
    },   
    beds: {
      keywords: ["beds"],
      checked: false
    },  
    remdesivir: {
      keywords: ["remdesivir"],
      checked: false
    }, 
    fabiflu: {
      keywords: ["fabiflu"],
      checked: false
    },
    tocilizumab: {
      keywords: ["tocilizumab"],
      checked: false
    },
    plasma: {
      keywords: ["plasma"],
      checked: false
    },
    tiffin: {
      keywords: ["tiffin"],
      checked: false
    }
  };
  const excludeKeywords = {
    needed: {
      keywords: ['needed'],
      checked: true,
    },
    required: {
      keywords: ['required'],
      checked: true,
    }
  };

  let links = LocalStorage.getItem(STORAGE_KEY.generated_links, []);
  let popularCityLinks = [];

  $: alsoSearchFor, inputs, checkboxes, generatePopularCityLinks();
  //'re-run this code whenever any of the referenced values change'

  function generatePopularCityLinks() {
    popularCityLinks = POPULAR_CITIES.map(city => {
      return {
        city,
        href: generateLinkForCity(city)
      };
    });
  }

  function getAlsoSearchForString() {
    const keywords = Object.keys(alsoSearchFor).reduce((keywordsSoFar, item) => {
      if (alsoSearchFor[item].checked) {
        return keywordsSoFar.concat(alsoSearchFor[item].keywords);
      } else {
        return keywordsSoFar;
      }
    }, []);

    if (inputs.otherAlsoSearchFor) {
      keywords.push(inputs.otherAlsoSearchFor);
    }

    if (keywords.length > 0) {
      return `${keywords.join(" OR ")}`;
    } else {
      return "";
    }
  }

  function getExcludedKeywordsString() {
    const keywords = Object.keys(excludeKeywords).reduce((keywordsSoFar, item) => {
      if (excludeKeywords[item].checked) {
        return keywordsSoFar.concat(excludeKeywords[item].keywords);
      } else {
        return keywordsSoFar;
      }
    }, []);

    if (inputs.otherExcludedKeywords) {
      keywords.push(inputs.otherExcludedKeywords);
    }

    return keywords.map(keyword => `-"${keyword}"`).join(' ');
  }

  function generateLinkForCity(city) {
    const base = `https://facebook.com/search/posts/`;
    const params = new URLSearchParams();

    const query = [
      checkboxes.verifiedOnly && "verified",
      city.trim(),
      getAlsoSearchForString(),
      checkboxes.excludeUnverified && '-"not verified"',
      checkboxes.excludeUnverified && '-"unverified"',
      //getExcludedKeywordsString(),
    ]
      .filter(Boolean)
      .join(" ");

    params.set("q", query);
    //params.set("f", "live");

    if (checkboxes.nearMe) {
      params.set("lf", "on");
    }

    const link = `${base}?${params.toString()}`;

    return link;
  }

  function generate() {
    if (!inputs.cities) {
      alert("Please enter city name(s)");
      return;
    }

    const cities = inputs.cities
      .split(",")
      .map(city => city.trim())
      .filter(Boolean);

    links = cities.map(city => {
      return {
        city,
        href: generateLinkForCity(city)
      };
    });

    tick().then(() => {
      const firstItem = document.querySelector("#city-links li");

      if (firstItem) {
        firstItem.scrollIntoView();
        firstItem.focus();

        alert('Please check the Links section');
      }

      LocalStorage.setItem(STORAGE_KEY.generated_links, links);
    });
  }

  function clearSavedLinks() {
    links = [];

    LocalStorage.removeItem(STORAGE_KEY.generated_links);
  }
</script>
<style>
  * {
      box-sizing: border-box;
  }
  
  /* Header/logo Title */
  .header {
      padding: 80px;
      text-align: center;
      background: #4267B2     ;
      color: white;
      font-family: Lucida Sans Unicode;

  }

  /* Increase the font size of the heading */
  .header h1 {
      font-size: 40px;
  }

  main{
      width: 100%;
  }
  button{
        background-color: #ddd;
        border: none;
        color: black;
        padding: 10px 10px 10px 10px;
        text-align: center;
        text-decoration: none;
        display: inline-block;
        margin: 10px 10px;
        cursor: pointer;
        border-radius: 16px;
    }
#citylinks{
  background-color: #ddd;
        border: none;
        color: black;
        padding: 10px 10px 10px 10px;
        text-align: center;
        text-decoration: none;
        display: inline-block;
        margin: 10px 10px;
        cursor: pointer;
        border-radius: 16px;
}
#pass{
  font-style: italic;
}
</style>


<main>
<div class="header">
  <h1>COVID Help from Facebook!</h1>
  <h2>Some Suggestions:</h2>
  <p>Facebook Login is required</p>
  <p>For better accuracy, select only 1 option</p>

  <form on:submit|preventDefault={generate}>
    <div>
      <label for="cities">Enter City Name</label>
      
      <input type="text" bind:value={inputs.cities} id="cities"/>
    </div>

    <div id="checkboxes">
      Search for :
      {#each Object.keys(alsoSearchFor) as item (item)}
        <div class="row">
          <input type="checkbox" bind:checked={alsoSearchFor[item].checked} id={`alsoSearchFor-${item}`} />
          <label for={`alsoSearchFor-${item}`} id="chee">{capitalCase(item)}</label>
        </div>
      {/each}

      <div>
        <label for="alsoSearchFor-other">Other:</label>
        <input type="text" bind:value={inputs.otherAlsoSearchFor} id="alsoSearchFor-other" />
      </div>

    </div>
    
    <div>
      <button>Generate</button>
    </div>
  </form>

  {#if links.length > 0}
    <h2>Your Generated Links</h2>

    <ol id="city-links">
      {#each links as link (link.href)}
        <a href={link.href} target="_blank" rel="noopener noreferrer" id="citylinks">{capitalCase(link.city)}</a>
      {/each}
    </ol>

    <button on:click={clearSavedLinks}>Clear saved links</button>
  {/if}

  <div id="pass">
    <h3>This too shall pass.</h3>
  </div>
  </div>

  
</main>

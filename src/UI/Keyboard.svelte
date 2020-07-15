<script>
    import {onMount} from 'svelte';
    import {cubicInOut} from 'svelte/easing';
    import {scale} from 'svelte/transition';
    import chords from '../data/chords';
    import options from '../data/options';
    export let keys;
    let player;
    let sound = null;
    let chord = [];
    let outputChord = null;
    let key = null;
    let playChord = null;
    let selectedKey = '';
    let selectedType = '';
    let inversion = 0;


    const setKey = k => {
        const selected = keys.findIndex(ky => ky.id === k);
        keys[selected].active = !keys[selected].active;
        key = keys[selected];
        const audio = new Audio(key.sound)
        if(key.active) {
            audio.play();
            chord.push(key);
        } else {
            const index = chord.findIndex(c => c.id === key.id)
            chord.splice(index, 1);
        }
        analyseChord(chord);
        playChord = chord;
    }

    const analyseChord = ch => {
        outputChord = null; // reset output chord
        let chrd = [...ch]; // copy chord
        chrd.sort((a,b) => a.id > b.id); // sort by id
        let notes = chrd.map(i => i.name); // extract notes
        notes = [...new Set(notes)];    // remove duplicates
        chords.forEach((chd, i) => { // loop through chords
            const {data} = chd // extract variations
            const found = data.find(i => JSON.stringify(i) == JSON.stringify(notes)) // find matches
            if(found) {
                outputChord = chords[i].name; // set chord output name
            }
        })
    }

    const play = () => {
        if(playChord) {
            playChord.forEach((el, i) => {
                i = new Audio(el.sound)
                i.play()
            });
        }
    }

    const clear = () => {
        const ks = [...keys];
        ks.forEach(k => {
            k.active = false;
        });
        chord = [];
        outputChord = null;
        key = null;
        keys = ks;
    }

    const reset = () => {
        clear();
        selectedKey = '';
        selectedType = '';
    }

    const chooseChord = (inv) => {
        clear()
        if(selectedKey && selectedType) {
            if(selectedKey.length > 1) { // remove alias
                selectedKey = selectedKey.substr(0,2)
            }
            let ks = [...keys]; // copy keys
            const selected = selectedKey + ' ' + selectedType; // concat key and type
            const found = chords.find(ch => ch.name.includes(selected)); //find the chord
            let notes = found.data[inv]; // create notes array
            notes.forEach((note, i) => { // loop through notes and assign key
                const fnd = ks.filter(k => k.name === note)
                if(i === 0) {
                    fnd[0].active = true;
                    chord.push(fnd[0])
                } else {
                    if(chord[0].id < fnd[0].id) {
                        fnd[0].active = true;
                        chord.push(fnd[0]);
                    } else {
                        fnd[1].active = true;
                        chord.push(fnd[1])
                    }
                }
            })
            keys = ks; // replace keys with copied
            playChord = chord; // set chord
            play() // play chord
            analyseChord(chord); // pass chord to analyse function
        }
    }

    // preload audio
    onMount(() => {
        const sounds = keys.map(i => i.sound);
        sounds.forEach((sound, i) => {
            const audio = new Audio(sound);
        });
    })
</script>

<div class="chord">
    {#if outputChord}
    <h3 transition:scale={{duration: 300, easing: cubicInOut}}>{outputChord}</h3>
    <button on:click={play} transition:scale={{duration: 300, easing: cubicInOut}}>Play</button>
    {/if}
</div>



<div class="keyboard">
    {#each keys as key}
    <div class="keyboard__key {key.active ? 'keyboard__key--active' : ''} {key.class === 'black' ? 'keyboard__key--black' : ''} " data-name="{key.name}" data-alt="{key.alt}" on:click={() => setKey(key.id)} >
    </div>
    {/each}
</div>

<div class="controls">
    <div>
        <select bind:value={selectedKey} on:blur={() => chooseChord(inversion)}>
            <option value="">-- Key --</option>
            {#each options[0].keys as key}
                <option value="{key}">{key}</option>
            {/each}
        </select>
        <select bind:value={selectedType} disabled={selectedKey === ''} on:blur={() => chooseChord(inversion)}>
            <option value="">-- Type --</option>
            {#each options[1].types as type}
                <option value="{type}">{type}</option>
            {/each}
        </select>
    </div>
    <button on:click={reset}>Clear Keys</button>
</div>


<style>
    .keyboard {
        display: flex;
        height: 301px;
        margin: 10px auto 0 auto;
        justify-content: center;
        box-sizing: border-box;
        margin-bottom: 50px;
    }

    .keyboard__key {
        height: 180px;
        border-right:2px solid #000;
        border-left:2px solid #000;
        border-top: 1px solid #000;
        width: 83px;
        height: 300px;
        display: flex;
        justify-content: center;
        align-items: center;
        font-size: 1.6em;
        text-align: center;
        font-family: 'noto';
        position: relative;
        cursor: pointer;
        border-bottom: 1px solid #000;
        border-radius: 0 0 6px 6px;
        background: #fff;
        transition: background .1s;
    }

    .keyboard__key::after {
        content: attr(data-name);
        position: absolute;
        bottom: 20px;
        display: block;
    }

    .keyboard__key--black {
        width:0px;
        height: 0px;
        border: none;
    }

    .keyboard__key--black:after {
        content: attr(data-name) '' attr(data-alt);
        position: absolute;
        word-break: break-all;
        top: 1px;
        display: flex;
        justify-content: center;
        align-items: flex-end;
        padding-bottom: 20px;
        background: #000;
        color:#fff;
        height: 170px;
        width: 60px;
        z-index: 2;
        padding:8px;
        box-sizing: border-box;
        border: 4px solid #000;
        border-top: none;
        box-sizing: border-box;
        border-radius: 0 0 6px 6px;
        transition: background .1s;
    }

    .keyboard__key--active {
        background: #f1eb93;
    }

    .keyboard__key--active.keyboard__key--black:after{
        background: #f1eb93;
        color: #000;
    }

    .chord {
        display: flex;
        justify-content: center;
        align-items: center;
        width:100%;
        height: 150px;
        font-size: 3em;
        font-family: 'noto';
        color:#fff;
    }

    .chord button {
        cursor: pointer;
        margin-left: 10px;
        position: relative;
        border: 3px solid #fff;
        color:#000;
        width:50px;
        height: 50px;
        background: transparent;
        border-radius: 50%;
        transition: border .5s;
    }

    .chord button:after {
        content: '';
        position: absolute;
        top:50%;
        left: 50%;
        transform: translateX(-40%) translateY(-50%);
        width:20px;
        height: 20px;
        background: #fff;
        clip-path: polygon(0 0, 0% 100%, 100% 50%);
        transition: background .5s;
    }

    .chord button:hover {
        border-color: #f1eb93;
    }

    .chord button:hover:after {
        background: #f1eb93;
    }

    .controls {
        display: flex;
        justify-content: space-between;
        margin-bottom: 50px;
    }

    .controls button {
        color: #fff;
        font-size: 1em;
        cursor: pointer;
        border: 1px solid #fff;
        background: transparent;
        padding: 7px 12px;
        border-radius: 6px;
        margin-right: 4px;
        transition: all .3s;
    }

    .controls button:hover {
        color:#f1eb93;
        border-color: #f1eb93;
    }

    .controls button:active {
        transform: scale(0.92)
    }

    .controls select {
        font-size: 1em;
        -webkit-appearance: none;
        background: transparent;
        border: 1px solid #fff;
        color: #fff;
        text-align: center;
        margin-right: 12px;
        padding: 7px 12px;
        border-radius: 6px;
        transition: all .3s;
    }

    .controls select:first-of-type {
        width:100px;
    }

    .controls select:disabled {
        border-color: #777;
        color: #777;
    }
</style>
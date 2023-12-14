<template>
    <div class="min-h-screen bg-gray-200 p-4">
        <div v-if="!data.setupDone" class="flex space-x-4">
            <div>
                <label class="block mb-2 text-sm font-medium text-gray-900">Region</label>
                <select @change="selectedRegion($event)" class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5">
                    <option selected>Select a region</option>
                    <option v-for="region in this.regions" :key="region.id">{{ region.name }}</option>
                </select>
            </div>

            <div v-for="(player, index) in data.players" :key="index">
                <label class="block mb-2 text-sm font-medium text-gray-900">Player {{ index + 1 }}</label>
                <input v-model="player.name" type="text" class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5" />
            </div>

            <div>
                <label class="block mb-2 text-sm font-medium text-gray-900">Add player</label>
                <button type="button" @click="addPlayer">
                    <svg xmlns="http://www.w3.org/2000/svg" height="40" width="40" viewBox="0 0 512 512"><path fill="#0080ff" d="M256 512A256 256 0 1 0 256 0a256 256 0 1 0 0 512zM232 344V280H168c-13.3 0-24-10.7-24-24s10.7-24 24-24h64V168c0-13.3 10.7-24 24-24s24 10.7 24 24v64h64c13.3 0 24 10.7 24 24s-10.7 24-24 24H280v64c0 13.3-10.7 24-24 24s-24-10.7-24-24z"/></svg>
                </button>
            </div>

            <div>
                <label class="block mb-2 text-sm font-medium text-gray-900">Setup</label>
                <button type="button" @click="completeSetup" class="text-white bg-blue-700 hover:bg-blue-800 focus:ring-4 focus:ring-blue-300 font-medium rounded-lg text-sm px-5 py-2.5 me-2 mb-2">Ready</button>
            </div>

            <div>
                <label class="block mb-2 text-sm font-medium text-gray-900">Restore session</label>
                <button type="button" @click="restoreSession" class="text-white bg-blue-700 hover:bg-blue-800 focus:ring-4 focus:ring-blue-300 font-medium rounded-lg text-sm px-5 py-2.5 me-2 mb-2">Retrieve</button>
            </div>

            <div>
                <label class="block mb-2 text-sm font-medium text-gray-900">Import Data</label>
                <input type="file" accept=".json" @change="importData" class="text-white font-medium rounded-lg text-sm px-5 py-2.5 me-2 mb-2"/>
            </div>
        </div>

        <div v-if="data.setupDone" class="grid lg:grid-cols-4 grid-cols-1 space-x-6">
            <div class="relative lg:col-span-4">
                <table class="w-full text-sm text-left rtl:text-right text-gray-500">
                    <thead class="text-xs text-gray-900 uppercase">
                    <tr>
                        <th scope="col" class="md:px-6 px-1 py-3">
                            Location
                        </th>
                        <th v-for="(player, index) in data.players" :key="index" scope="col" class="md:px-6 px-1 py-3">
                            {{ player.name }}
                        </th>
                        <th scope="col" class="md:px-6 px-1 py-3">
                            Nicknames
                        </th>
                    </tr>
                    </thead>
                    <tbody>
                    <tr v-for="(catchingLocation, index) in data.catchingLocations" :key="index" class="bg-white">
                        <td class="md:px-6 px-1 py-4">
                            <select v-model="catchingLocation.name" @change="onLocationSelected($event, index)" class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5">
                                <option selected disabled>Select location</option>
                                <option v-for="location in locations" :key="location.id" :value="location.name"
                                        :disabled="data.catchingLocations.some(catchLoc => catchLoc.name === location.name)">
                                    {{ location.name }}
                                </option>
                            </select>
                        </td>
                        <td v-for="(player, playerIndex) in data.players" :key="playerIndex" class="md:px-6 px-1 py-4">
                            <template v-if="pokemonImagesForLocations[`${playerIndex}-${catchingLocation.name}`]">
                                <div class="w-20 h-20 overflow-hidden flex items-center justify-center">
                                    <img @click="getPokemonEvolutions(pokemonImagesForLocations[`${playerIndex}-${catchingLocation.name}`][0], playerIndex, index)" :src="pokemonImagesForLocations[`${playerIndex}-${catchingLocation.name}`]" alt="Pokemon Image" class="max-w-full max-h-full cursor-pointer"/>
                                </div>
                            </template>
                            <template v-else>
                                <button @click="openModal(playerIndex)" type="button">
                                    <svg xmlns="http://www.w3.org/2000/svg" height="40" width="40" viewBox="0 0 512 512"><path fill="#0080ff" d="M256 512A256 256 0 1 0 256 0a256 256 0 1 0 0 512zM232 344V280H168c-13.3 0-24-10.7-24-24s10.7-24 24-24h64V168c0-13.3 10.7-24 24-24s24 10.7 24 24v64h64c13.3 0 24 10.7 24 24s-10.7 24-24 24H280v64c0 13.3-10.7 24-24 24s-24-10.7-24-24z"/></svg>
                                </button>
                            </template>
                        </td>


                        <td class="md:px-6 px-1 py-4">
                            <template v-for="(player, playerIndex) in data.players" :key="playerIndex">
                                <span v-for="(pokemonId, index) in catchingLocation.pokemons" :key="index">
                                    <span v-if="player.pokemons.party.some(pokemon => pokemon.id === pokemonId) || player.pokemons.box.some(pokemon => pokemon.id === pokemonId) || player.pokemons.fainted.some(pokemon => pokemon.id === pokemonId)">
                                        {{ getPlayerPokemonNickname(player, pokemonId) }}
                                        <span v-if="index < catchingLocation.pokemons.length - 1"> & </span>
                                    </span>
                                </span>
                            </template>
                        </td>

                        <td class="md:px-6 px-1 py-4">
                            <select v-model="catchingLocation.storage" @change="onStorageChanged($event, index)" class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5">
                                <option :value="'party'" :selected="catchingLocation.storage === 'party'" :disabled="data.players[0].pokemons.party.length >= 6">Party</option>
                                <option :value="'box'" :selected="catchingLocation.storage === 'box'">Box</option>
                                <option :value="'fainted'" :selected="catchingLocation.storage === 'fainted'">Fainted</option>
                            </select>
                        </td>
                    </tr>
                    </tbody>
                </table>
                <div class="md:px-6 px-4 py-4">
                    <button type="button" @click="addCatchingLocation" class="text-white bg-blue-700 hover:bg-blue-800 focus:ring-4 focus:ring-blue-300 font-medium rounded-lg text-sm px-5 py-2.5 me-2 mb-2">Add location</button>
                </div>
            </div>

            <div class="flex flex-col">
                <div class="flex-1">
                    <table class="w-full text-sm text-left rtl:text-right text-gray-500">
                        <caption class="p-5 text-lg font-semibold text-left rtl:text-right text-gray-900 bg-white">
                            Party
                        </caption>
                        <tbody>
                        <tr class="bg-white mb-2 border-b-2 border-gray-200">
                            <td v-for="(player, playerIndex) in this.data.players" :key="playerIndex" class="px-4 py-4 w-1/6">
                                <template v-for="index in 6" :key="index">
                                    <div v-if="player.pokemons.party.length >= index" class="flex items-center space-x-2">
                                        <div class="flex items-center">
                                            <template v-if="pokemonImagesForLocations[`${playerIndex}-${player.pokemons.party[index - 1].location}`]">
                                                <div class="w-20 h-20 overflow-hidden flex items-center justify-center">
                                                    <img :src="pokemonImagesForLocations[`${playerIndex}-${player.pokemons.party[index - 1].location}`]" alt="Pokemon Image" class="max-w-full max-h-full">
                                                </div>
                                            </template>
                                            <span class="font-bold">{{ player.pokemons.party[index - 1].nickname || player.pokemons.party[index - 1].pokemon }}</span>
                                        </div>
                                    </div>
                                    <div v-else class="flex items-center justify-center w-10 h-10">
                                        <svg xmlns="http://www.w3.org/2000/svg" fill="#ED4956" viewBox="0 0 24 24" stroke="#fff" class="w-10 h-10">
                                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 21c4.97 0 9-4.03 9-9s-4.03-9-9-9-9 4.03-9 9 4.03 9 9 9zm0-2a7 7 0 100-14 7 7 0 000 14z"/>
                                            <circle cx="12" cy="12" r="5" fill="#fff"/>
                                        </svg>
                                    </div>
                                </template>
                            </td>
                        </tr>

                        </tbody>
                    </table>
                </div>
            </div>

            <div class="relative lg:col-start-3">
                <table class="w-full text-sm text-left rtl:text-right text-gray-500 dark:text-gray-400 bg-cover bg-center" style="background-image: url('/box_background.png');">
                    <tbody>
                    <tr v-for="(row, rowIndex) in computedTableData" :key="rowIndex">
                        <td v-for="(cell, colIndex) in row" :key="colIndex" class="px-4 py-4 w-16 h-16">
                            <template v-if="cell">
                                <div class="w-12 h-12 overflow-hidden flex items-center justify-center">
                                    <img :src="pokemonImagesForLocations[`${cell.playerIndex}-${cell.location}`]" alt="Pokemon Image" class="max-w-full max-h-full" />
                                </div>
                            </template>
                        </td>
                    </tr>
                    </tbody>
                </table>
            </div>

            <div class="px-6">
                <button type="button" @click="exportData" class="text-white bg-blue-700 hover:bg-blue-800 focus:ring-4 focus:ring-blue-300 font-medium rounded-lg text-sm px-5 py-2.5 me-2 mb-2">Export Data</button>
            </div>
        </div>

        <div v-if="isOpen">
            <div class="fixed z-10 inset-0 overflow-y-auto" aria-labelledby="modal-title" role="dialog" aria-modal="true">
                <div class="flex items-end justify-center min-h-screen pt-4 px-4 pb-20 text-center sm:block sm:p-0">
                    <div class="fixed inset-0 bg-gray-500 bg-opacity-75 transition-opacity"
                         aria-hidden="true"></div>
                    <span class="hidden sm:inline-block sm:align-middle sm:h-screen"
                          aria-hidden="true">&#8203;</span>
                    <div
                        class="relative inline-block align-bottom bg-white rounded-lg px-4 pt-5 pb-4 text-left overflow-hidden shadow-xl transform transition-all sm:my-8 sm:align-middle sm:max-w-lg sm:w-full sm:p-6">
                        <div>
                            <div class="mt-3  sm:mt-5">
                                <h3 class="text-lg text-center leading-6 font-medium text-gray-900"
                                    id="modal-title">Add pokemon</h3>
                                <div>
                                    <label class="block mb-2 text-sm font-medium text-gray-900">Pokemon</label>
                                    <multiselect
                                        v-model="selectedPokemon"
                                        :options="pokemons"
                                        label="name"
                                        placeholder="Select a pokemon"
                                        track-by="name"
                                        :allow-empty="false"
                                        :searchable="true"
                                        class="w-full p-2.5 overflow-x-auto border border-gray-300 focus:ring-blue-500 focus:border-blue-500 rounded-lg"
                                    ></multiselect>
                                </div>
                                <div>
                                    <label class="block mb-2 text-sm font-medium text-gray-900">Nickname</label>
                                    <input v-model="givenNickname" type="text" class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5" />
                                </div>
                                <div class="mt-2">
                                    <label class="relative inline-flex items-center cursor-pointer">
                                        <input type="checkbox" v-model="pokemonCaught" class="sr-only peer" checked>
                                        <div class="w-11 h-6 bg-gray-200 rounded-full peer peer-focus:ring-4 peer-focus:ring-blue-300 peer-checked:after:translate-x-full rtl:peer-checked:after:-translate-x-full peer-checked:after:border-white after:content-[''] after:absolute after:top-0.5 after:start-[2px] after:bg-white after:border-gray-300 after:border after:rounded-full after:h-5 after:w-5 after:transition-all peer-checked:bg-blue-600"></div>
                                        <span class="ms-3 text-sm font-medium text-gray-900">Caught?</span>
                                    </label>
                                </div>
                            </div>
                        </div>
                        <div class="mt-5 sm:mt-6 sm:grid sm:grid-cols-2 sm:gap-3 sm:grid-flow-row-dense">
                            <button @click="addPokemon()" type="button" class="w-full inline-flex justify-center rounded-md border border-transparent shadow-sm px-4 py-2 bg-indigo-600 text-base font-medium text-white hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 sm:col-start-2 sm:text-sm">Add</button>
                            <button type="button" class="mt-3 w-full inline-flex justify-center rounded-md border border-gray-300 shadow-sm px-4 py-2 bg-white text-base font-medium text-gray-700 hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 sm:mt-0 sm:col-start-1 sm:text-sm" @click="this.isOpen = false">Cancel</button>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <div v-if="evolutionModalOpen">
            <div class="fixed z-10 inset-0 overflow-y-auto" aria-labelledby="modal-title" role="dialog" aria-modal="true">
                <div class="flex items-end justify-center min-h-screen pt-4 px-4 pb-20 text-center sm:block sm:p-0">
                    <div class="fixed inset-0 bg-gray-500 bg-opacity-75 transition-opacity"
                         aria-hidden="true"></div>
                    <span class="hidden sm:inline-block sm:align-middle sm:h-screen"
                          aria-hidden="true">&#8203;</span>
                    <div class="relative inline-block align-bottom bg-white rounded-lg px-4 pt-5 pb-4 text-left overflow-hidden shadow-xl transform transition-all sm:my-8 sm:align-middle sm:max-w-lg sm:w-full sm:p-6">
                        <div>
                            <div class="mt-3 sm:mt-5">
                                <h3 class="text-lg text-center leading-6 font-medium text-gray-900" id="modal-title">Choose evolution</h3>
                            </div>
                            <div class="flex flex-wrap justify-center items-center space-x-10">
                                <div v-for="(evolution, index) in evolutionsList" :key="index" class="mb-4">
                                    <img :src="evolution.image" :alt="evolution.name + ' image'" @click="changePokemonEvolution(index, evolution.name)" class="cursor-pointer max-w-full">
                                </div>
                            </div>
                        </div>
                        <div class="mt-5 sm:mt-6 sm:grid sm:gap-3 sm:grid-flow-row-dense">
                            <button type="button" class="mt-3 w-full inline-flex justify-center rounded-md border border-gray-300 shadow-sm px-4 py-2 bg-white text-base font-medium text-gray-700 hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 sm:mt-0 sm:col-start-1 sm:text-sm" @click="this.evolutionModalOpen = false">Cancel</button>
                        </div>
                    </div>

                </div>
            </div>
        </div>
    </div>
</template>

<script>
import Multiselect from "vue-multiselect";
export default {
    name: "App",

    components: { Multiselect },

    data() {
        return {
            data: {
                region: null,
                setupDone: false,
                idCounter: 0,
                players: [
                    {
                        name: '',
                        pokemons: {
                            party: [],
                            box: [],
                            fainted: [],
                        },
                    }
                ],
                catchingLocations: [
                    {
                        name: null,
                        pokemons: [],
                        storage: null
                    }
                ],
            },

            regions: [],
            locations: [],
            pokemons: [],
            selectedPokemon: null,
            selectedPlayer: null,
            selectedLocation: null,
            locationsIndex: null,
            selectedStorage: null,
            givenNickname: null,
            pokemonCaught: true,
            evolveChain: null,
            evolutionsList: [],
            evolutionModalOpen: false,
            selectedEvolutionPokemon: null,
            isOpen: false,
            pokemonData: localStorage.pokemonData ? JSON.parse(localStorage.pokemonData) : null,
        };
    },

    created() {
        this.fetchRegions()
        this.fetchPokemon()
    },

    computed: {
        filteredLocations() {
            return this.locations.filter(location =>
                this.data.catchingLocations.some(catchingLocation => catchingLocation.name === location.name)
            );
        },

        pokemonImagesForLocations() {
            const pokemonImages = {};

            this.data.players.forEach((player, playerIndex) => {
                this.data.catchingLocations.forEach((catchingLocation) => {
                    const pokemonsInLocation = catchingLocation.pokemons;

                    const matchingPokemon = pokemonsInLocation
                        .map((pokemonId) => {
                            // Check if the Pokemon is in the party
                            const foundInParty = player.pokemons.party.find((pokemon) => pokemon.id === pokemonId);
                            if (foundInParty) {
                                return foundInParty.image;
                            }

                            // Check if the Pokemon is in the box
                            const foundInBox = player.pokemons.box.find((pokemon) => pokemon.id === pokemonId);
                            if (foundInBox) {
                                return foundInBox.image;
                            }

                            // Check if the Pokemon is in the fainted storage type
                            const foundInFainted = player.pokemons.fainted.find((pokemon) => pokemon.id === pokemonId);
                            return foundInFainted ? foundInFainted.image : null;
                        })
                        .filter((image) => image !== null);

                    if (matchingPokemon.length > 0) {
                        pokemonImages[`${playerIndex}-${catchingLocation.name}`] = matchingPokemon;
                    }
                });
            });
            return pokemonImages;
        },

        computedTableData() {
            const columns = 6;
            const rows = 5;
            const tableData = [];

            // Generate empty cells
            for (let rowIndex = 0; rowIndex < rows; rowIndex++) {
                const row = Array(columns).fill(null);
                tableData.push(row);
            }

            // Replace empty cells with Pokémon from the box
            let remainingPokemons = [];
            this.data.players.forEach((player, playerIndex) => {
                player.pokemons.box.forEach((pokemon) => {
                    const key = `${playerIndex}-${pokemon.location}`;
                    if (!remainingPokemons.includes(key)) {
                        const emptyCellIndex = tableData.flat().indexOf(null);
                        if (emptyCellIndex !== -1) {
                            const rowIndex = Math.floor(emptyCellIndex / columns);
                            const colIndex = emptyCellIndex % columns;
                            tableData[rowIndex][colIndex] = { ...pokemon, playerIndex };
                            remainingPokemons.push(key);
                        }
                    }
                });
            });

            return tableData;
        }
    },

    watch: {
        'data': {
            handler(newData) {
                localStorage.pokemonData = JSON.stringify(newData);
            },
            deep: true,
        },
    },

    methods: {
        fetchRegions() {
            this.axios
                .get(`https://pokeapi.co/api/v2/region`)
                .then((response) => {
                    this.regions = response.data.results;
                })
                .catch((error) => {
                    console.error(error);
                });
        },

        fetchPokemon() {
            this.axios
                .get(`https://pokeapi.co/api/v2/pokemon?limit=2000`)
                .then((response) => {
                    this.pokemons = response.data.results.map(pokemon => ({
                        name: pokemon.name,
                    }));
                })
                .catch((error) => {
                    console.error(error);
                });
        },

        async getPokemonDetails() {
            try {
                const response = await this.axios.get(`https://pokeapi.co/api/v2/pokemon/${this.selectedPokemon.name}`);
                const animatedFrontDefault = response.data.sprites.versions['generation-v']['black-white'].animated.front_default;
                const normalFrontDefault = response.data.sprites.front_default;

                return animatedFrontDefault || normalFrontDefault;
            } catch (error) {
                console.error(error);
                return null;
            }
        },

        getPlayerPokemonNickname(player, pokemonId) {
            const partyPokemon = player.pokemons.party.find(pokemon => pokemon.id === pokemonId);
            const boxPokemon = player.pokemons.box.find(pokemon => pokemon.id === pokemonId);
            const faintedPokemon = player.pokemons.fainted.find(pokemon => pokemon.id === pokemonId);

            return partyPokemon ? partyPokemon.nickname || partyPokemon.pokemon :
                (boxPokemon ? boxPokemon.nickname || boxPokemon.pokemon :
                    (faintedPokemon ? faintedPokemon.nickname || faintedPokemon.pokemon : ''));
        },

        async addPokemon() {
            const pokemonImage = await this.getPokemonDetails();

            const storageType = this.data.players[this.selectedPlayer].pokemons.party.length < 6 ? 'party' : 'box';

            const newPokemon = {
                id: this.data.idCounter,
                pokemon: this.selectedPokemon.name,
                nickname: this.givenNickname,
                location: this.selectedLocation,
                image: pokemonImage,
                caught: this.pokemonCaught,
                storage: storageType
            };

            this.data.players[this.selectedPlayer].pokemons[storageType].push(newPokemon);

            const catchLocationIndex = this.data.catchingLocations.findIndex(location => location.name === this.selectedLocation);

            this.data.catchingLocations[catchLocationIndex].storage = storageType;

            this.data.catchingLocations[catchLocationIndex].pokemons.push(this.data.idCounter);

            this.data.idCounter++;
            this.selectedPokemon = null;
            this.givenNickname = null;
            this.isOpen = false;
        },

        async getPokemonEvolutions(id, playerIndex, locationIndex) {
            this.selectedPlayer = playerIndex;
            this.locationsIndex = locationIndex;
            this.selectedStorage = this.data.catchingLocations[locationIndex];

            this.evolutionsList = [];

            const match = id.match(/\/(\d+)\.(gif|png)$/);
            const pokemonId = match ? match[1] : null;

            try {
                // Fetch the species details
                const speciesResponse = await this.axios.get(`https://pokeapi.co/api/v2/pokemon-species/${pokemonId}`);
                const evolutionChainUrl = speciesResponse.data.evolution_chain.url;

                // Fetch the evolution chain details
                const evolutionChainResponse = await this.axios.get(evolutionChainUrl);

                // Extract and fetch images of the evolution Pokémon
                const extractEvolutionDetails = (evolution) => {
                    const evolutionSpecies = evolution.species.name;
                    const evolutionUrl = `https://pokeapi.co/api/v2/pokemon/${evolutionSpecies}`;

                    // Fetch Pokémon details to get the image
                    this.axios.get(evolutionUrl)
                        .then((response) => {
                            const sprites = response.data.sprites;

                            const imageUrl = sprites.versions['generation-v']['black-white'].animated.front_default;

                            const fallbackImageUrl = sprites.front_default;

                            this.evolutionsList.push({ name: evolutionSpecies, image: imageUrl || fallbackImageUrl });
                        })
                        .catch((error) => {
                            console.error(error);
                        });

                    // Recursively process further evolutions
                    if (evolution.evolves_to && evolution.evolves_to.length > 0) {
                        evolution.evolves_to.forEach(extractEvolutionDetails);
                    }
                };

                // Start processing the evolution chain
                const chain = evolutionChainResponse.data.chain;
                extractEvolutionDetails(chain);

            } catch (error) {
                console.error(error);
            }

            this.evolutionModalOpen = true;
        },

        changePokemonEvolution(index, EvolutionPokemon) {
            // Get the selected player
            const selectedPlayer = this.data.players[this.selectedPlayer];

            // Determine the storage type of the Pokémon (assuming it's party for illustration)
            const storageType = this.selectedStorage.storage; // Update this based on your logic

            // Find the selected Pokémon in the player's data
            const selectedPokemonIndex = selectedPlayer.pokemons[storageType].findIndex(pokemon => pokemon.id === this.data.catchingLocations[this.locationsIndex].pokemons[this.selectedPlayer]);

            if (selectedPokemonIndex !== -1) {
                selectedPlayer.pokemons[storageType][selectedPokemonIndex].pokemon = EvolutionPokemon;
                selectedPlayer.pokemons[storageType][selectedPokemonIndex].image = this.evolutionsList[index].image;
            }

            this.evolutionModalOpen = false;
        },

        selectedRegion(event) {
            this.data.region = event.target.value;
        },

        addPlayer() {
            this.data.players.push({
                id: this.data.players.length,
                name: '',
                pokemons: {
                    party: [],
                    box: [],
                    fainted: []
                }
            });
        },

        completeSetup() {
            this.axios
                .get(`https://pokeapi.co/api/v2/region/${this.data.region}`)
                .then((response) => {
                    this.locations = response.data.locations.map(location => ({
                        name: location.name
                    }));
                    this.data.setupDone = true;
                })
                .catch((error) => {
                    console.error(error);
                });
        },

        openModal(index) {
            this.selectedPlayer = index;
            this.isOpen = true;
        },

        onLocationSelected(event, index) {
            this.selectedLocation = event.target.value;
            this.data.catchingLocations[index].name = this.selectedLocation;
        },

        onStorageChanged(event, locationIndex) {
            const newStorageType = event.target.value;

            // Update storage in catchingLocation
            this.data.catchingLocations[locationIndex].storage = newStorageType;

            // Iterate through each player and update storage for the given location
            this.data.players.forEach(player => {
                const movedPokemonIds = this.data.catchingLocations[locationIndex].pokemons;

                // Add moved Pokemon to the new storage type (box, party, or fainted)
                player.pokemons[newStorageType] = [
                    ...player.pokemons[newStorageType],
                    ...movedPokemonIds.map(pokemonId => {
                        const existingPokemon =
                            player.pokemons.party.find(p => p.id === pokemonId) ||
                            player.pokemons.box.find(p => p.id === pokemonId) ||
                            player.pokemons.fainted.find(p => p.id === pokemonId);

                        return existingPokemon ? { ...existingPokemon, storage: newStorageType } : null;
                    })
                ].filter(Boolean); // Remove potential null entries from the array

                // Remove moved Pokemon from the current storage (party, box, or fainted)
                const currentStorageType =
                    newStorageType === 'party' ? ['box', 'fainted'] : newStorageType === 'box' ? ['party', 'fainted'] : ['party', 'box'];

                currentStorageType.forEach(storage => {
                    player.pokemons[storage] = player.pokemons[storage].filter(pokemon =>
                        !movedPokemonIds.includes(pokemon.id)
                    );
                });
            });
        },

        addCatchingLocation() {
            this.data.catchingLocations.push({
                name: null,
                pokemons: [],
                storage: null
            });
        },

        restoreSession() {
            this.data = this.pokemonData;

            this.completeSetup()

            const lastCatchingLocation = this.data.catchingLocations[this.data.catchingLocations.length - 1];
            if (lastCatchingLocation) {
                this.selectedLocation = lastCatchingLocation.name;
            }
        },

        exportData() {
            const jsonData = JSON.stringify(this.data);
            const blob = new Blob([jsonData], { type: 'application/json' });
            const link = document.createElement('a');
            link.href = URL.createObjectURL(blob);
            link.download = 'exported_data.json';
            link.click();
        },
        importData(event) {
            const file = event.target.files[0];
            if (file) {
                const reader = new FileReader();
                reader.onload = (e) => {
                    try {
                        // Assuming this.data is the variable you want to update
                        this.data = JSON.parse(e.target.result);
                        this.completeSetup()
                        alert('Data imported successfully!');
                    } catch (error) {
                        console.error('Error parsing JSON:', error);
                        alert('Error importing data. Please check the file format.');
                    }
                };
                reader.readAsText(file);
            }

            console.log(this.data.region, this.locations)


            const lastCatchingLocation = this.data.catchingLocations[this.data.catchingLocations.length - 1];
            if (lastCatchingLocation) {
                this.selectedLocation = lastCatchingLocation.name;
            }
        },
    }
};
</script>
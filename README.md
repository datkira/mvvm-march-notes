# Model View ViewModel - MVVM

### View

- The _View_ is the only touching point for a user with your application.
- A user will interact with your _View_ that will trigger _ViewController_ methods depending on events such as mouse
  movements, key presses etc.
- It should be used only for displaying data and triggering events passed from the _ViewController_. (React.Component)
- The _View_ is not only used for a user input but also for displaying output — results of some actions.

=> This way, we’re keeping our components reusable and easy to test. With the help of MobX, we’ll turn React.Component
into reactive component that will observe any changes and automatically update itself accordingly.

```javascript
import React from "react";
import PokemonList from "./UI/PokemonList";
import PokemonForm from "./UI/PokemonForm";

class PokemonView extends React.Component {
  render() {
    const {
      pokemons,
      pokemonImage,
      pokemonName,
      randomizePokemon,
      setPokemonName,
      addPokemon,
      removePokemon,
      shouldDisableSubmit,
    } = this.props;

    return (
      <React.Fragment>
        <PokemonForm
          image={pokemonImage}
          onInputChange={setPokemonName}
          inputValue={pokemonName}
          randomize={randomizePokemon}
          onSubmit={addPokemon}
          shouldDisableSubmit={shouldDisableSubmit}
        />
        <PokemonList removePokemon={removePokemon} pokemons={pokemons} />
      </React.Fragment>
    );
  }
}

export default PokemonView;
```

### ViewModel

- It can be a React.Component => easy
- The _ViewModel_ is just a regular JavaScript class it can be easily reused anywhere with UI tailored differently.
- Every dependency needed by the `ViewModel` will be injected through the constructor, thus making it easy to test.
  The `ViewModel` is interacting directly with the Model and whenever the _ViewModel_ updates it, all changes will be
  automatically reflected back to the _View_.

```javascript
class PokemonViewModel {
  constructor(pokemonStore) {
    this.store = pokemonStore;
  }

  getPokemons() {
    return this.store.getPokemons();
  }

  addPokemon(pokemon) {
    this.store.addPokemon(pokemon);
  }

  removePokemon(pokemon) {
    this.store.removePokemon(pokemon);
  }
}

export default PokemonViewModel;
```

### Model

The Model is acting as a data source i.e. global store for the application. It composes all data from the network layer,
databases, services and serve them easily. It shouldn’t have any other logic except one that actually updates a model
and does not have any side effects.

![mvvm machine](assets/machine.png)

```javascript
import { observable, action } from "mobx";
import uuid from "uuid/v4";

class PokemonModel {
  @observable pokemons = [];

  @action addPokemon(pokemon) {
    this.pokemons.push({
      id: uuid(),
      ...pokemon,
    });
  }

  @action removePokemon(pokemon) {
    this.pokemons.remove(pokemon);
  }

  @action clearAll() {
    this.pokemons.clear();
  }

  getPokemons() {
    return this.pokemons;
  }
}

export default PokemonModel;
```

#### Provider

One component that is not part of the MVVM, but we’ll use it to glue everything together is called Provider. This
component will instantiate _ViewModel_ and provide all needed dependency to it. Furthermore, instance of the _ViewModel_ is
passed through props to the _ViewController_ component. Provider should be clean, without any logic as its purpose is just
to wire up everything.

```javascript
import React from "react";
import { inject } from "mobx-react";
import PokemonController from "./PokemonController";
import PokemonViewModel from "./PokemonViewModel";
import RootStore from "../../models/RootStore";

@inject(RootStore.type.POKEMON_MODEL)
class PokemonProvider extends React.Component {
  constructor(props) {
    super(props);
    const pokemonModel = props[RootStore.type.POKEMON_MODEL];
    this.viewModel = new PokemonViewModel(pokemonModel);
  }

  render() {
    return <PokemonController viewModel={this.viewModel} />;
  }
}

export default PokemonProvider;
```

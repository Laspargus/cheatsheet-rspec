# GEMS UTILES

## EN DEV + TEST
- rspec-rails: rspec pour rails \o/
- rubocop: rubocop ça check la syntaxe de ton code :)

## EN TEST
- factory_bot_rails: Permet de créer facilement des instance de modèle en database.
- faker: Permet de générer de fausses données de test.
- nyan-cat-formatter: C’est mignon les nyan cat pendant les tests :)
- shoulda-matchers: Permet de vérifier les validations facilement dans les tests.

# SYNTAXE

## DESCRIBE/CONTEXT
Les deux font la même chose et permettent de définir un “contexte” de test. Et donc de séparer nos tests pour les rendre plus lisibles.

On utilisera describe plus pour préciser des noms de classe/méthode:

```ruby 
describe MaClass do
  describe "#new" do
    …
  end
end
```

On utilisera context pour décrire un contexte plus global:
```ruby
describe MaClass do
  context "when the user isn't logged in" do
    …
  end
end
```

## IT
it permet de faire un test. Il est conseillé de ne pas mettre “should” dedans. Par exemple:

```ruby
it "has 3 elements" do
  …
end
```

on notera l’utilisation de la troisième personne du singulier. Pourquoi fait on cela ? Outre le fait de rendre les tests plus proches de l’anglais, c’est parce qu’après, rspec permet de générer une “doc” de cette façon:
```ruby
rspec --format doc
```


## EXPECT
Ça permet de tester quelque chose. Une liste de matchers peut être trouvé ici: https://relishapp.com/rspec/rspec-expectations/v/3-7/docs/built-in-matchers

```ruby
expect([1,2,3]).to include(2)
```

## LET/LET!
let et son ami let! vous permettent de définir une variable qui sera utilisable dans vos tests. Les let sont scope par describe/context.

Un exemple d’utilisation:

```ruby
context "this will work" do
  let(:ma_var) { "yop" }

  it "checks let" do
    expect(ma_var).to eq("yop")
  end
end

context "this will not work" do
  it "checks let" do
    expect(ma_var).to eq("yop")
  end
end
```

La différence entre let et let! c’est que let est lazy et ne sera instancié que la première fois que la variable est appelée tandis que let! instancie la variable avant le test. C’est particulièrement pratique quand les let! instancie des objets en database.

## BEFORE

before permet de définir un block qui sera exécuté avant chaque test. Par exemple:

```ruby
before do
  Turtle.create(name: 'silvie', color: 'green')
end

it "has one turtle in db" do
  expect(Turtle.count).to eq(1)
end
```


## SUBJECT
C’est un peu comme un let ça permet de définir le “sujet” du test. On l’invoque plus tard dans son test en faisant subject

```ruby
subject do
  get :index
end

it "has an array of turtles" do
  subject
  expect(JSON.parse(response.body)).to be_a(Array)
end
```

## FactoryBot
FactoryBot est une gem pour nous aider à définir des factories qui permettent de générer des objets en DB dans nos tests.

On l’installe en ajoutant dans spec/rails_helper dans le block de configuration:

```config.include FactoryBot::Syntax::Methods```

Les factories sont définies dans spec/factories et ressemblent à:

```ruby
FactoryBot.define do
  factory :turtle do
    name { Faker::OnePiece.character }
    color { %w(blue green pink yellow).sample }
  end
end
```

On les invoque plus tard dans nos tests de cette façon:

```ruby
# créer une tortue en db.
create(:turtle)

# créer une tortue en db en spécifiant certains de ses attributs
create(:turtle, name: 'Léonardo')

# créer plusieurs instances (12 ici)
create_list(:turtle, 12)
```

##ShouldaMatchers
Permet de tester les validation des modèles (entre autre).


On l’installe en ajoutant à la fin du spec/rails_helper

```ruby
Shoulda::Matchers.configure do |config|
  config.integrate do |with|
    # Choose a test framework:
    with.test_framework :rspec
    with.library :rails
  end
end
```

Par exemple si le modèle défini une validation de présence:


validates :name, presence: true
on la test de cette façon:

```ruby
it { should validate_presence_of(:name) }
```

La liste des helpers de tests se trouve ici: https://github.com/thoughtbot/shoulda-matchers

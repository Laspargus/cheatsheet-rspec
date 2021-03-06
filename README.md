# GEMS UTILES

## EN DEV + TEST
- rspec-rails: rspec pour rails \o/
- rubocop: rubocop ça check la syntaxe de ton code :)

```
rails g rspec:install
Running via Spring preloader in process 24305
  create  .rspec
  create  spec
  create  spec/spec_helper.rb
  create  spec/rails_helper.rb
```

Pour configurer rspec avec Factory Bot, il faut éditer le fichier spec/rails_helper.rb 

### Remove this line if you're not using ActiveRecord or ActiveRecord fixtures
```
config.fixture_path = "#{::Rails.root}/spec/fixtures"
```
Mets en commentaire la ligne non commentée puis ajoute juste après la ligne suivante :
```
config.include FactoryBot::Syntax::Methods
```

Pour installer shouldamatcher, il faut également éditer le fichier rails_helper et ajouter :
```
Shoulda::Matchers.configure do |config|
  config.integrate do |with|
    with.test_framework :rspec
    with.library :rails
  end
end
```


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


## Les différences entre Before, Subject, Let et Let!
(avec exemples et utilisation de Factory)

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

La différence entre let et let! c’est que let est lazy et ne sera instancié que la première fois que la variable est appelée tandis que let! instancie la variable avant le test.C’est particulièrement pratique quand les let! instancie des objets en database.

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

Si l'instance pose problème, on peut ajouter ce bloc pour avoir le détail lors de la commande rspec

```ruby
@stay_other_tenant_other_studio = FactoryBot.create(:stay).tap { |e| p e.valid?; p e.errors }
```

Il se déclare généralement après le describe et va permettre d'appeler une varibale dans tous les tests qui suivent. 

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

Exemple de test de modèle avec utilisation d'un subject

```ruby
require 'rails_helper'

RSpec.describe Auction, :type => :model do
  subject { described_class.new }

  it "is valid with valid attributes" do
    subject.title = "Anything"
    expect(subject).to be_valid
  end

  it "is not valid without a title" do
    subject.title = nil
    expect(subject).to_not be_valid
  end

  it "is not valid without a description"
    subject.description = nil
    expect(subject).to_not be_valid
   end

end
```
Parfois il est nécessaire de combiner un subject avec un let. 

Voici un exemple : 

```ruby 
let(:seller) { User.new(:email => "jane@doe.com", :password => "pw1234",
                       :password_confirmation => "pw1234") }
subject {
  described_class.new(title: "Anything", description: "Lorem ipsum",
                    start_date: DateTime.now, end_date: DateTime.now + 1.week,
                    seller: seller)
}
```
La plupart du temps, subject est utilisé pour renvoyer une instance de classe. 


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

## ShouldaMatchers
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



# Les différents types de test

## Test de Modèle

### Tester la base de donnée

```ruby
require 'rails_helper'

# RSpec.describe Product, type: :model do
# end

RSpec.describe Product, type: :model do
  
    describe 'Database' do
      it { is_expected.to have_db_column(:id).of_type(:integer)}
      it { is_expected.to have_db_column(:description).of_type(:text)}
      it { is_expected.to have_db_column(:image).of_type(:string)}
      it { is_expected.to have_db_column(:price).of_type(:float)}
      it { is_expected.to have_db_column(:title).of_type(:string)}
      it { is_expected.to have_db_column(:updated_at).of_type(:datetime)}
      it { is_expected.to have_db_column(:updated_at).of_type(:datetime)}
      it {is_expected.to have_db_column(:category_id).of_type(:integer)}
    end
end
```


### Tester les attributs d'une instance 

```ruby
  describe 'Product attributes' do
    let(:product)  { build(:product) }

    context 'Price' do
      it { expect(product.price).to be > 0 }
    end

    context 'Image https' do 
      it { expect(product.image).to match(/https:/)}
    end
  end
```


### Tester les association d'un modèle

```ruby
   ## Shoulda Matcher : is_expected.to ==> should 

  describe 'Product Association' do
    it { is_expected.to belong_to(:category) }
    it { is_expected.to have_many(:selections) }
    it { should have_many(:carts) }
    it { should have_many(:order_products) }
    it { should have_many(:orders) }
  end
```


### Tester les Validates

```ruby
  ## Shoulda Matcher : is_expected.to ==> should 

  describe 'Product Validates' do
    
    it { is_expected.to validate_presence_of(:price) }
    it { is_expected.to validate_presence_of(:title) }
    it { is_expected.to validate_presence_of(:description) }
    it { should validate_presence_of(:image ) }
    it { should validate_presence_of(:category_id ) }
  end
```


On peut également tester les validates en créant une instance et en passant successivement chaque attribut obligatoire à nil. 

Exemple : 

```ruby 
 ########## ATTRIBUTES ############

  describe 'Product attributes' do
    let(:product) { build(:product) }

    it "is valid without any errors" do
      expect(product.errors).to be_empty 
    end

    it "is valid with valid attributes" do
      expect(product).to be_valid
    end

    context "product should not be valid" do
      it "is not valid with missing all attributes" do
       expect(Product.new).to_not be_valid
      end

      it "is not valid with missing price attributes" do
        product.price = nil
        expect(product).to_not be_valid
      end

      it "is not valid with missing descritpion attributes" do
        product.description = nil
        expect(product).to_not be_valid
      end

      it "is not valid with missing titile  attributes" do
        product.title = nil
        expect(product).to_not be_valid
      end
    end
```

## Test de Controller

##Cypress

```$npx cypress open```
Permet d'ouvrir Cypress en mode interactif

```$npx cypress run```
Permet d'ouvrir cypress dans le terminal sans visuel (headless mode)





### Liens Utiles

#### Test de modèle
https://semaphoreci.com/community/tutorials/how-to-test-rails-models-with-rspec

#### Test de controleur
https://kolosek.com/rspec-controller-test/

#### Let vs Before
https://kolosek.com/rspec-let-vs-before/

#### Better Spec
http://www.betterspecs.org/#subject

#### CheatSheet Factory Boat
https://devhints.io/factory_bot

#### Built In Matcher
https://relishapp.com/rspec/rspec-expectations/v/3-7/docs/built-in-matchers

#### CheatSheet Rspec
https://devhints.io/rspec-rails

#### Set up Cypress
https://docs.cypress.io/guides/getting-started/installing-cypress.html#Installing

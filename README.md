#DESCRIBE/CONTEXT
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

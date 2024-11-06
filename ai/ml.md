* dados, modelos e aprendizado

# Notação

1. se `X` e `Y` são conjuntos/categorias, `f:X->Y` denota função/funtor entre eles
2. se `X` é conjunto/categoria e `x` é seu elemento/objeto, `x in X` denota a relação de continência
7. `Cat` é a categoria de todas as categorias e functores
3. `R` denota o corpo dos números reais ou a categoria tendo `R` como único elemento
4. `Vec` é a categoria dos espaços vetoriais reais de dimensão finita
5. `dim: Vec -> R` é o functor que a cada espaço vetorial retorna sua dimensão
6. se `C` e `W` são categorias, `prod(V,W)` ou `VxW` denota o produto entre elas
7. se `V` e `W` são espaços vetoriais, `VxW` e `V+W` denotam seu produto cartesiano e soma direta, respectivamente
8. dado inteiro `n in N`, `Rn` denota `RxRx...xR`
9. se `X,Y in C` são objetos de uma categoria `C`, então `f:X~>Y` denota um isomorfismo entre elas, e `X(~f)Y` denota a relação de equivalência associada.


# Definições

> **Def.** Um *espaço amostral linear paramétrico* é um espaço vetorial `V` isomorfo a `R(N+D)~R(N)xR(D)`. Fala-se que:
> 1. `R(N)` é o *espaço amostral linear*, tendo elementos `x in R(N)` chamados de *dados lineares de dimensão `N`*
> 2. `R(D)` é o *espaço de parâmetros*.

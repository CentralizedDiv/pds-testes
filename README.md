# Documentação de 3 exemplos reais de testes

## Vue JS

O [Vue JS](https://github.com/vuejs/vue) é um framework javascript usado para construir interfaces web. Uma das funcionalidades do Vue é o server side rendering, que é a renderização, e hidratação de html feita por um servidor node. A seguir, 3 testes de unidade presentes no repositório do VueJS.

### [1 - Renderizando HTML a partir de um template](https://github.com/vuejs/vue/blob/dev/test/ssr/ssr-template.spec.js#L44)

```javascript
// Imports
import { createRenderer } from '../../packages/vue-server-renderer'
const { renderToString } = createRenderer()

// Definição do template
const defaultTemplate = `<html><head></head><body><!--vue-ssr-outlet--></body></html>`

// Teste
describe('SSR: template option', () => {
  it('renderToString', done => {
    const renderer = createRenderer({
      template: defaultTemplate
    })

    const context = {
      head: '<meta name="viewport" content="width=device-width">',
      styles: '<style>h1 { color: red }</style>',
      state: { a: 1 }
    }

    renderer.renderToString(new Vue({
      template: '<div>hi</div>'
    }), context, (err, res) => {
      expect(err).toBeNull()
      expect(res).toContain(
        `<html><head>${context.head}${context.styles}</head><body>` +
        `<div data-server-rendered="true">hi</div>` +
        `<script>window.__INITIAL_STATE__={"a":1}</script>` +
        `</body></html>`
      )
      done()
    })
  })
```

Esse teste verifica se a função renderToString, que está presente no objeto renderer retorna corretamente um html encapsulado pelo template que é passado. Nas primeiras linhas é definido um template padrão (defaultTemplate) contendo as tags padrões do html (\<html\>, \<head\> e \<body\>) e no teste este template é passado como parâmetro na hora de criar o renderer. Sendo assim, no assert (aqui chamado de expect) é verificado se o template a ser renderizado (\<div>hi\</div> ) é corretamente renderizado dentro do template padrão e com as variáveis necessárias para que o Vue funcione, como window.\_\_INITIAL_STATE\_\_, já que isso foi passado no content (variável state).

### [2 - E2E na página de exemplos](https://github.com/vuejs/vue/blob/dev/test/e2e/specs/modal.js)

Os testes e2e no repositório do Vue são rodados usando Selenium. Um arquivo de inicialização, que não tem sentido mostrar aqui, chama a função abaixo dentro do contexto necessário.

```javascript
module.exports = {
  'modal': function (browser) {
    browser
    .url('http://localhost:8080/examples/modal/')
      .waitForElementVisible('#app', 1000)
      .assert.elementNotPresent('.modal-mask')
      .click('#show-modal')
      .assert.elementPresent('.modal-mask')
      .assert.elementPresent('.modal-wrapper')
      .assert.elementPresent('.modal-container')
      .waitFor(50)
      .assert.cssClassPresent('.modal-mask', 'modal-enter-active')
      .waitFor(300)
      .assert.cssClassNotPresent('.modal-mask', 'modal-enter-active')
      .assert.containsText('.modal-header h3', 'custom header')
      .assert.containsText('.modal-body', 'default body')
      .assert.containsText('.modal-footer', 'default footer')
      .click('.modal-default-button')
      // should have transition
      .assert.elementPresent('.modal-mask')
      .waitFor(50)
      .assert.cssClassPresent('.modal-mask', 'modal-leave-active')
      .waitFor(300)
      .assert.elementNotPresent('.modal-mask')
      .end()
  }
}
```

Achei interessante mostrar um dos testes E2E que eles tem para a página de [exemplos de código](https://br.vuejs.org/v2/examples/), nessa página existem vários exemplos de como implementar funcionalidades comuns numa interface, como um modal.

No teste acima, a página web é aberta e testada verificando se o modal não está aberto no início e apartir dai algumas interações e outros asserts são feitos, achei bem legal o fato deles testarem até mesmo a página de exemplos e me ajudou em testes pessoas, já que nunca tinha usado o selenium até pouco tempo.


### [3 - Funcionalidade de _filter with pipes_](https://github.com/vuejs/vue/blob/dev/test/unit/features/filter/filter.spec.js#L18)

```javascript
  it('chained usage', () => {
    const vm = new Vue({
      template: '<div>{{ msg | upper | reverse }}</div>',
      data: {
        msg: 'hi'
      },
      filters: {
        upper: v => v.toUpperCase(),
        reverse: v => v.split('').reverse().join('')
      }
    }).$mount()
    expect(vm.$el.textContent).toBe('IH')
  }) 
```

Escolhi este teste unitário porque apesar de simples, ele testa uma das funcionalidades mais legais e úteis do Vue quando se trata de pequenos projetos. O __filter__ nada mais é do que um jeito fácil e rápido de aplicar uma função a um parâmetro num contexto de html. A sintaxe é parecida com o __pipe__ da linha de comando do linux, onde você pode concatenar várias funções e o resultado da primeira é passado como parâmetro para a segunda.

No teste acima, uma string 'hi' é modificada por dois filtros: __upper__, que converte a string para maíusculas; e reverse, que inverte a string. Sendo assim, o teste pega o conteúdo do html e testa com 'IH', que é o resultado esperado após a concatenação destes dois filtros.

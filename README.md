# eTemplate2 --> LitElement / lit-html
This repo explores the idea of transforming eTempalte2 using LitElement and lit-html

## The "problem(s)"
* eTempalte2 relys a lot on now obsolte technolegies like jQuery, jQueryUI and libraries based on theses
* Layout in eTemplate2 is "messy" as theming, layout in apps and styling of individual widgets mess with each other
* eTemplate2 always re-renders / rebuilds the whole DOM, not just changed parts (unless you only tell the changed widget to update)
* [custom TS/JS dependency resultion and execution order evaluation, combined with ancient LAB.js loading](https://github.com/EGroupware/lit/blob/main/TsJsLoading.md)

## Possible solution: use webcomponents, LitElement, lit-html and a component library

Idea is to use [webcomponents](https://www.webcomponents.org/introduction) specially [LitElement](https://lit-element.polymer-project.org/guide) and [lit-html](https://lit-html.polymer-project.org/guide) to replace or renew most of our current client-side eTempalte2 code *gradually*. 

> The word *gradually* need to be stressed, as we only have limited ressources and a huge code-base. So picking one of the big frameworks and rewriting everything based on that framework is out of scope.

Current state of my (Ralf) ideas:
* transform existing eTemplate2 / .xet files via a script into a LitElement (including transforming our id based data-binding into expressions in a lit-html template literal. This process can happen on the fly when the template get loaded and be cached by browser and server-side to still support eTemplate format and customization.
* wrap existing eT2 widgets as a webcompenent / LitElement
* use formdata event / support to collect data again when eT2 submits
* gradually replace existing wrapped eT2 widgets with native webcompenents / LitElement based widgets (mainly from a single component library plus complementing it with missing components)

So an existing eTemplate template like:
```
<overlay>
  <template id="myapp.edit.sub">
    <textbox label="Foo label" id="foo"/>
    <textbox label="Bar label" id="bar"/>
    <button label="Submit" id="button[save]"/>
  </template>
  <template id="myapp.edit">
    <groupbox>
      <caption label="Please fill in..."/>
      <template id="myadd.edit.sub"/>
    </groupbox>
  </template>
</overlay>
```
will be transformed to something like:
```
import { LitElement, html } from 'lit-element';

class Et2TemplateMyappEditSub extends Et2Template // extends LitElement 
{
  render() {
    return html`
    <div id="...">
      <et2-textbox label="${lang('Foo label'}" value=${content.foo}></et2-textbox>
      <et2-textbox label="${lang('Bar label'}" value=${content.bar}></et2-textbox>
      <et2-button label="${lang('Submit'}"/></et2-button>
    </div>
    `;
  }
}
customElements.define('et2-template-myapp-edit-sub', Et2TemplateMyappEditSub);

class Et2TemplateMyappEdit extends Et2Template // extends LitElement 
{
  render() {
    return html`
    <div id="...">
      <et2-groupbox >
         <et2-caption label="${lang('Please fill in...')}"></et2-caption>
         <et2-template-myapp-edit-sub value="${content}"></et2-template-myapp-edit-sub/>
      </et2-groupbox>
    </div>
    `;
  }
}
customElements.define('et2-template-myapp-edit', Et2TemplateMyappEdit);
```
And etemplate2 class would use:
```
let tpl = window.createElement('et2-template-myapp-edit');
tpl.set_value(content);
tpl.set_select_options(sel_options);
```
If ```tpl.set_value(changed_content)``` would be called later only DOM parts that need modification would be re-renders thanks to lit-html.

### Templates
I (Nathan) think we want the templates in the .xet file to be transformed to something we can let WebComponents render more directly.  Ex:
```
  <et2-template id="myapp.edit.sub">
    <et2-textbox label="${lang('Foo label'}" value=${content.foo}></et2-textbox>
    <et2-textbox label="${lang('Bar label'}" value=${content.bar}></et2-textbox>
    <et2-button label="${lang('Submit'}"/></et2-button>
  </et2-template>
```
Then have et2-template (widget) use built in stuff to handle it all:
```
import { LitElement, html } from 'lit-element';

class Et2Template // extends LitElement 
{
  constructor() {
      super();
      let template = document.getElementById(this.id); // Actually, load from cached file
      let templateContent = template.content;

      const shadowRoot = this.attachShadow({mode: 'open'})
        .appendChild(templateContent.cloneNode(true));
    }
  }
}
customElements.define('et2-template', Et2Template);
```
Note no need for render, since WebComponents [recursively] handles the children.  We may need to add some stuff, but do not need to add the children there.
https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_templates_and_slots


## Resources around webcompenents, LitElement, lit-html, ...
- https://webcomponents.dev/blog/all-the-ways-to-make-a-web-component/
- https://dzone.com/articles/web-component-solutions-a-comparison
- https://www.nexmo.com/legacy-blog/2020/05/20/web-components-tools-a-comparison
- https://lit-element.readthedocs.io/en/v0.6.4/docs/templates/databinding/
- https://github.com/gitaarik/lit-state#readme (Small libary implementing a global state with changes reflected in the DOM)

## Component libraries using LitElement
- https://github.com/carbon-design-system/carbon-custom-elements (IBM: https://www.carbondesignsystem.com/)
- https://github.com/material-components/material-components-web-components (Google)
- https://vaadin.com/components (commercial, thought it seems the code is open sourced)
- https://www.htmlelements.com/demos/ (nice, but little above the base libary is usable in an open source project)
- https://project-awesome.org/web-padawan/awesome-lit-html (see component libraries or inidividual components, not all use LitElement!)

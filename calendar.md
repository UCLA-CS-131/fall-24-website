---
layout: page
title: Course Calendar
description: Listing of course modules and topics.
---

# Course Calendar

<form id="filter-form">
  Filter by:
  <input type="checkbox" id="Lecture" name="Lecture" value="Lecture" checked />
  <label for="Lecture">Lecture</label>&nbsp;
  <input type="checkbox" id="Section" name="Section" value="Section" checked />
  <label for="Section">Section</label>&nbsp;
  <input type="checkbox" id="Posted" name="Posted" value="Posted" checked />
  <label for="Posted">Posted</label>&nbsp;
  <input type="checkbox" id="Due" name="Due" value="Due" checked />
  <label for="Due">Due</label>&nbsp;
  <input type="checkbox" id="Exam" name="Exam" value="Exam" checked />
  <label for="Exam">Exam</label>&nbsp;
</form>

{% for module in site.modules %}
{{ module }}
{% endfor %}

<script>
const groupArrayBy2 = (array) => {
  const result = [];
  for (let i = 0; i + 1 < array.length; i += 2) {
    result.push([array[i], array[i + 1]]);
  }
  return result;
};

const LABELS = ['Lecture', 'Section', 'Posted', 'Due', 'Exam'];
const modules = document.getElementsByClassName('module');
const moduleDls = [...modules].map(module => module.children[0]);
const items = [...moduleDls].map(module => groupArrayBy2([...module.children]));

const ddApplyClasses = (item, classDecider) => {
  for (const [dt, dd] of item) {
    dt.className = ''; // necessary ... for some reason?
    dd.className = '';
    const newClass = classDecider(dd);
    dt.className = newClass;
    dd.className = newClass;
  }
}

const ddPredicateBuilder = (labels) => {
  return (ddNode) => {
    console.log(ddNode.innerText)
    return labels
      // this toUpperCase is a hack to only catch the label
      .map(label => ddNode.innerText.includes(label.toUpperCase()))
      .some(v => v)
  }
}

const ddClassDecider = (predicate) => {
  return (ddNode) => predicate(ddNode) ? '' : 'd-none'
}

const rerenderItems = () => {
  const activeLabels =
    LABELS
      .map(label => document.getElementById(label).checked ? label : '')
      .filter(label => label !== '');
  console.log(activeLabels);

  for (const item of items) {
    ddApplyClasses(item, ddClassDecider(ddPredicateBuilder(activeLabels)))
  }
}

const filterForm = document.getElementById('filter-form');
filterForm.addEventListener('change', rerenderItems);
</script>
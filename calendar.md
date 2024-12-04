---
layout: page
title: Course Calendar
description: Listing of course modules and topics.
nav_exclude: true
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

<!-- *[Matt](https://matthewwang.me) says*: yes, the filtering leaves the day even if there's nothing there. Working on it! -->


{% for module in site.modules %}
{{ module }}
{% endfor %}

<script>
const LABELS = ['Lecture', 'Section', 'Posted', 'Due', 'Exam'];
const modules = document.getElementsByClassName('module');
const moduleDls = [...modules].map(module => module.children[0]);
const items = [...moduleDls].map(module => module.children);

const ddApplyClasses = (children, classDecider) => {
  for (const child of children){
    if (child.nodeName != 'DD') {
      continue;
    }
    child.className = ''; // necessary ... for some reason?
    const newClass = classDecider(child);
    child.className = newClass;
  }
}

const ddPredicateBuilder = (labels) => {
  return (ddNode) => {
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

  for (const item of items) {
    ddApplyClasses(item, ddClassDecider(ddPredicateBuilder(activeLabels)))
  }
}

const filterForm = document.getElementById('filter-form');
filterForm.addEventListener('change', rerenderItems);
</script>
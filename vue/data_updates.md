# How to receive updates on data changes during initialization of a vue.js component

1. Define properties of the component
```javascript
props: {
    type_prop: Object,
    required_prop: true
}
```

2. Define the data object in the component
```javascript
data() {
  return {
    type: this.type_prop,
    required: this.required_prop
  }
}
```

3. Define an event to be emitted in mounted hook
```javascript
mounted() {
  this.$emit('data-updated', { type: this.type, required: this.required });
}
```

4. Listen for the event in the parent component
```javascript
<child-component @data-updated="updateData"></child-component>
```

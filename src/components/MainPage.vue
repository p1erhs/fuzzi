<template>
  <div class="container">
    <project-name v-bind:project-name="projectName" v-model="projectName"/>
    <section class="project-variables">
      <form v-on:submit.prevent>
        <h4 @click="swapVariable">
          <span v-if="variables.variableType === 'input'">Input</span>
          <span v-if="variables.variableType === 'output'">Output</span> 
          variables (click)
        </h4>
        <label for="variable-name">Name:</label>
        <input type="text" id="variable-name" v-model="variables.newVariable.name">
        <label for="variable-fuzzy-areas-count">Number of fuzzyAreas:</label>
        <input type="text" id="variable-fuzzy-areas-count" v-model="variables.newVariable.fuzzyAreasCount">
        <br>
        <label for="fuzzy-area-range-start">Range:</label>
        <input type="number" step="any" id="fuzzy-area-range-start" v-model="variables.newVariable.start">
        <label for="fuzzy-area-range-end">Range:</label>
        <input type="number" step="any" id="fuzzy-area-range-end" v-model="variables.newVariable.end">
        <button @click="addVariable">Add variable</button>
      </form>

      <ul class="input-variables">
        <h4>Input variables: </h4>
        <li v-for="input in variables.inputs" :key="input.name">
          <p>{{input.name}} goes from <strong>{{input.start}} to {{input.end}}</strong></p>
          <p>FuzzyAreas: <span v-for="fuzzyArea in input.fuzzyAreas" :key="fuzzyArea.name">{{fuzzyArea.name}} </span></p>
          <section class="add-fuzzy-area" v-if="input.fuzzyAreasCount > input.fuzzyAreas.length">
            <h5>Add new input variable fuzzyArea:</h5>
            <form v-on:submit.prevent>
              <label for="fuzzy-area-name">Name:</label>
              <input type="text" id="fuzzy-area-name" v-model="fuzzyAreas.newFuzzyArea.name">
              <br>
              <label for="fuzzy-area-type">Type:</label>
              <select id="fuzzy-area-type" v-model="fuzzyAreas.newFuzzyArea.type">
                <option v-for="fuzzyArea in fuzzyAreas.types" v-bind:value="fuzzyArea" :key="fuzzyArea.name">{{fuzzyArea.name}}</option> 
              </select>
              <label for="fuzzy-area-value">Value:</label>
              <span v-for="(range, index) in fuzzyAreas.newFuzzyArea.type.ranges" :key="index"> 
                <input v-model="fuzzyAreas.newFuzzyArea.type.ranges[index]" type="number" step="any" id="fuzzyAreaValue">
              </span>
              <button @click="addFuzzyArea(input)">Add FuzzyArea</button>
            </form>
          </section>
        </li>
      </ul>
      <ul class="output-variables">
        <h4>Output variables: </h4>
        <li v-for="output in variables.outputs" :key="output.name">
          <p>{{output.name}} goes from <strong>{{output.start}} to {{output.end}}</strong></p>
          <p>FuzzyAreas: <span v-for="fuzzyArea in output.fuzzyAreas" :key="fuzzyArea.name">{{fuzzyArea.name}} </span></p>
          <section class="add-fuzzy-area" v-if="output.fuzzyAreasCount > output.fuzzyAreas.length">
            <h5>Add new output variable fuzzyArea:</h5>
            <form v-on:submit.prevent>
              <label for="fuzzy-area-name">Name:</label>
              <input type="text" id="fuzzy-area-name" v-model="fuzzyAreas.newFuzzyArea.name">
              <label for="fuzzy-area-type">Type:</label>
              <select id="fuzzy-area-type" v-model="fuzzyAreas.newFuzzyArea.type">
                <option v-for="fuzzyArea in fuzzyAreas.types" v-bind:value="fuzzyArea" :key="fuzzyArea.name">{{fuzzyArea.name}}</option> 
              </select>
              <label for="fuzzy-area-value">Value:</label>
              <div v-for="(range, index) in fuzzyAreas.newFuzzyArea.type.ranges" :key="index">
                <input v-model="fuzzyAreas.newFuzzyArea.type.ranges[index]" type="number" step="any" id="fuzzyAreaValue">
              </div>
              <button @click="addFuzzyArea(output)">Add FuzzyArea</button>
            </form>
          </section>
        </li>
      </ul>
    </section>
    
    <section id="examples-data">
      <h2><strong>Provide example for your variables: </strong></h2>
      <form v-on:submit.prevent>
        <div v-for="input in variables.inputs" class="variable-data" :key="input.name">
          <label for="example-value">{{input.name}}</label>
          <input type="number" step="any" id="example-value" v-model="input.example">
          <span class="contain-level" v-for="fuzzyArea in input.fuzzyAreas" :key="fuzzyArea.name">{{fuzzyArea.type.value(fuzzyArea.type.ranges, input.example)}}</span>
        </div>
        <!-- <div v-for="output in variables.outputs" class="variable-data" :key="output.name">
          <label for="example-value">{{output.name}}</label>
          <input type="number" step="any" id="example-value" v-model="output.example">
          <span class="contain-level" v-for="fuzzyArea in output.fuzzyAreas" :key="fuzzyArea.name">{{fuzzyArea.type.value(fuzzyArea.type.ranges, output.example)}}</span>
        </div> -->
      </form>
    </section>

    <section id="rules" v-if="variables.inputs.length">
      <form v-on:submit.prevent>
          <span>Compose your new fuzzyArea: </span>
          <br>
          <label for="rule-name"></label>
          <input type="text" id="rule-name" v-model="rules.newRule.name">
          <span>IF</span>
          <span v-for="(input, index) in variables.inputs" :key="index">
            <span v-if="index" id="norm" @click="toggleNorm">{{rules.newRule.type}}</span>
            <select v-model="rules.newRule.fuzzyAreas.inputs[index]">
              <option v-for="(fuzzyArea, indexFuzzyArea) in input.fuzzyAreas" :key="indexFuzzyArea" v-bind:value="fuzzyArea">
                {{fuzzyArea.name}}
              </option>
            </select>
          </span>
          <span>THEN</span>
          <span v-for="(output, index) in variables.outputs" :key="index">
            <select v-model="rules.newRule.fuzzyAreas.output">
              <option v-for="(fuzzyArea, indexFuzzyArea) in output.fuzzyAreas" :key="indexFuzzyArea" v-bind:value="fuzzyArea">
                {{fuzzyArea.name}}
              </option>
            </select>
          </span>
        <button @click="createRule">Add new rule</button>
      </form>

      <ul class="rules-view">
        <li v-for="(rule, index) in rules.data" v-bind:key="rule + index">
          <span>{{rule.name}} has: {{rule.fuzzyAreas.output.name}} of: {{rule.result}}</span>
        </li>
      </ul>
      <button @click="getResult">Log result</button>

    </section>

  </div>
</template>



<script>

import config from '../config/MainPageConfig';

export default {
  name: 'MainPage',
  data() {
    return config;
  },
  methods: {
    getResult() {
      /* eslint max-len: ["error", { "code": 280 }] */
      // I must change this!!!
      // let res = { result: 0 };
      // this.rules.data.forEach((element) => {
      //   console.log(element);
      //   if (element.result > res.result) res = element;
      // });
      // console.log(res.fuzzyAreas.output.type.ranges[0]);
      // const data = [];
      // this.rules.data.forEach((element) => {
      //   // example = element.fuzzyAreas.output.type.ranges[0];
      //   let example = parseInt(element.fuzzyAreas.output.type.ranges[0], 10);
      //   while (element.result !== element.fuzzyAreas.output.type.value(element.fuzzyAreas.output.type.ranges, example)) {
      //     example += 0.1;
      //     console.log(example);
      //   }
      //   data.push(example);
      // });
      // console.log(data);
      console.log(this.defuzzification());
      // return data;
    },
    defuzzification() {
      const stepN = 1000;
      const starter = this.variables.outputs[0].start;
      let ender = this.variables.outputs[0].end;
      ender = parseInt(ender, 10);
      const step = (ender - starter) / (stepN - 1);
      // let uppersum = 0;
      // let lowersum = 0;
      let keep = 0;
      const values = [];
      const weightedvalues = [];
      // for (let u = starter; u < ender; u = step + u) {
      //   const compvalues = [];
      //   for (let i = 0; i < this.rules.data.length; i += 1) {
      //     for (let j = 0; j < i; j += 1) {
      //       const value = this.rules.data[i].fuzzyAreas.output.type.value(this.rules.data[i].fuzzyAreas.output.type.ranges, u);
      //       if (value > 0) {
      //         if (value < this.rules.data[j].result) {
      //           compvalues.push(value);
      //         } else {
      //           compvalues.push(this.rules.data[j].result);
      //         }
      //       }
      //     }
      //   }
      //   if (compvalues.length > 0) {
      //     keep = Math.max(...compvalues);
      //     values.push(keep);
      //     weightedvalues.push(keep * u);
      //   }
      // }
      // console.log(keep);
      // console.log(values);
      // console.log(weightedvalues);
      for (let i = 0; i < this.rules.data.length; i += 1) {
        const compvalues = [];
        for (let j = starter; j < ender; j = step + j) {
          const value = this.rules.data[i].fuzzyAreas.output.type.value(this.rules.data[i].fuzzyAreas.output.type.ranges, i);
          if (value > 0) {
            if (value < this.rules.data[i].result) {
              compvalues.push(value);
            } else {
              compvalues.push(this.rules.data[i].result);
            }
          }
        }
        if (compvalues.length > 0) {
          keep = Math.max(...compvalues);
          values.push(keep);
          weightedvalues.push(keep * i);
        }
      }
      const sumwv = weightedvalues.reduce((a, b) => a + b, 0);
      const sumv = values.reduce((a, b) => a + b, 0);
      console.log(sumwv);
      console.log(sumv);
      return sumwv / sumv;
      // for (let i = 0; i < this.rules.data.length; i += 1) {
      //   for (let j = starter; j < ender; j = step + j) {
      //     const value = this.rules.data[i].fuzzyAreas.output.type.value(this.rules.data[i].fuzzyAreas.output.type.ranges, i) - (this.rules.data[i].result + 1);
      //     if (value > 0) {
      //       if (value < this.rules.data[i].result) {
      //         uppersum += value * j;
      //         lowersum += value;
      //       } else {
      //         uppersum += this.rules.data[i].result * j;
      //         lowersum += this.rules.data[i].result;
      //       }
      //     }
      //   }
      // }
      // return (uppersum / lowersum);
    },
    toggleNorm() {
      let newNorm;
      if (this.rules.newRule.type === 'AND') newNorm = 'OR';
      else newNorm = 'AND';
      this.rules.newRule = {
        ...this.rules.newRule,
        type: newNorm,
      };
    },
    checkValue(rule) {
      const compareFunction = rule.type === 'AND' ? Math.min : Math.max;
      const inputs = rule.fuzzyAreas.inputs;
      const data = [];
      inputs.forEach((element, index) => {
        const example = this.variables.inputs[index].example;
        data.push(element.type.value(element.type.ranges, example));
      });
      const result = data.reduce((next, prev) => compareFunction(next, prev));
      return result;
    },
    // getImplication(rule) {
    //   let example = 0;
    //   const data = rule.result;
    //   const out = rule.fuzzyAreas.output;
    //   console.log(out.type.value(out.type.ranges, example));
    //   example += 1;
    //   console.log(data);
    //   while (out.type.value(out.type.ranges, example) !== result) {
    //     example += 1;
    //   }
    //   console.log(example);
    //   return example;
    // },
    createRule() {
      this.rules = {
        ...this.rules,
        data: [
          ...this.rules.data,
          {
            ...this.rules.newRule,
            fuzzyAreas: {
              inputs: [...this.rules.newRule.fuzzyAreas.inputs],
              output: { ...this.rules.newRule.fuzzyAreas.output },

            },
            result: this.checkValue(this.rules.newRule),
            // tslint:disable-next-line:max-line-length
            // implication: this.getImplication(this.rules.newRule),
          },
        ],
      };
    },
    addFuzzyArea(input) {
      input.fuzzyAreas.push({
        ...this.fuzzyAreas.newFuzzyArea,
        type: {
          ...this.fuzzyAreas.newFuzzyArea.type,
          ranges: [...this.fuzzyAreas.newFuzzyArea.type.ranges],
        },
      });
    },
    addVariable() {
      if (this.variables.variableType === 'input') {
        this.variables.inputs = [
          ...this.variables.inputs,
          { ...this.variables.newVariable, fuzzyAreas: [] },
        ];
      } else {
        this.variables.outputs = [
          ...this.variables.outputs,
          { ...this.variables.newVariable, fuzzyAreas: [] },
        ];
      }
    },
    swapVariable() {
      if (this.variables.variableType === 'input') {
        this.variables.variableType = 'output';
      } else {
        this.variables.variableType = 'input';
      }
    },
  },
};

</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  margin: 0 10px;
}

a {
  color: #42b983;
}

span {
  padding: 10px;
}
</style>
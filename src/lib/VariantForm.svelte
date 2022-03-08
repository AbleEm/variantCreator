<script>
import { onMount } from 'svelte';
import * as sampleInput from './inputs'

let variantForm

onMount(()=>{
    variantForm = document.getElementById('variantForm')
})

let options = sampleInput.options2
let levels = {
    get length(){
        return options.attributes.length
    } 
}

options.attributes.map((attribute,index)=>{
    levels[`level${index+1}`] = {
        length : attribute.properties.length,
        children : attribute.properties
    }
})

console.log("LEVELS",levels);

console.log("test", Object.keys(levels));

let modifiedVariants = []
sampleInput.variants2.map(combination =>{
if(combination.stock === "inactive") {
    let value = combination.attribute_values.map(item => item.value)
    modifiedVariants.push(value);
    }
})

let invalidCombiSearchFn = (inputArray,searchKey) => {
    let invalidCombinations = []
    inputArray.map((input)=> {
        if(input.find(item => item === searchKey)){
            invalidCombinations.push(input)
        }
    })
    console.log("INVALID", invalidCombinations);
    return invalidCombinations
}

let disableInput = (inputNode) => {
    let element = document.getElementById(inputNode)
    // @ts-ignore
    if (element.checked===true){
    // @ts-ignore
        element.checked = false
    }
    // @ts-ignore
    element.setAttribute("disabled", true)
}

// let getActiveSiblings = ()=> {

// }

let focusInput = (inputNode) => {
    let element = document.getElementById(inputNode)
    // @ts-ignore
    element.checked = true
}

let getOccurrence = (array, value)=> {
    let count = 0;
    array.forEach((v) => (v === value && count++));
    return count;
}

let formHandler= (event) => {

    let inputs = [...variantForm.querySelectorAll('input')]
    
    let selectedInputs = []
    inputs.map((input) => {
        if (input.checked == true) {
            selectedInputs.push(input)
        }
        if (input.hasAttribute("disabled")){
            input.removeAttribute("disabled")
        }
    })

    let modifiedInputs = selectedInputs.map(input => input.value)
    let unavailableCombinations = invalidCombiSearchFn(modifiedVariants,modifiedInputs[0])
    
    let getDisableElements = ()=>{
        let elementList = []
        unavailableCombinations.map((item)=> {
            if (item.includes(modifiedInputs[levels.length-2])){
                elementList.push(item)
                return elementList
            }
        })
        return elementList
    }

    let getParentAvailability = () => {
        let helper = unavailableCombinations.map(combi=> combi[levels.length-2])
        let finalEl = null
        helper.forEach((item)=>{
            let attributeOccurence = getOccurrence(helper,item)
            let childElementsCount = levels[`level${levels.length}`].length
            if (attributeOccurence === childElementsCount){
                finalEl = item
            }           
        })      
        return finalEl
    }
    if(getParentAvailability()){
        disableInput(getParentAvailability())
    }

    let disableElements = getDisableElements()

    if (disableElements.length>0){
        disableElements.forEach((input)=>{
            disableInput(input[input.length -1])
        })
    }
}


</script>

<form id="variantForm" on:change={formHandler}>
    {#each options.attributes as attribute, i}
        <div id={attribute.kind} style="display:flex; gap: 30px;">
            {#each attribute.properties as property,j}                
                <div>
                    <input type="radio" id={property} name={attribute.kind} value={property} checked={j===0? true : false}>
                    <label for={property}>{property}</label>
                </div>
            {/each}
        </div>
    {/each}
</form>
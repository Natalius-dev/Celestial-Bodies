<script>
    import Button from "../lib/Button.svelte"
    import { onMount } from "svelte";
    localStorage.setItem("SolarSystem.json", "")

    onMount(() => {
        // Select existing Star System
        const selectDOM = document.getElementById("selectSystems");
        const items = Object.keys({ ...localStorage });
        for(let i = 0; i < items.length; i++) {
            let option = document.createElement("option");
            option.value = items[i];
            option.innerHTML =  items[i];
            selectDOM.appendChild(option);
        }

        document.getElementById("open").onclick =
        function openSystem() {
            console.log(localStorage.getItem(selectDOM.value));
        }

        // Import new Star System
        const jsonErr = document.getElementById("jsonErr");
        let fileName;
        let result;
        let formatted;
        document.getElementById('import').onclick = function() {
        let files = document.getElementById('selectFiles').files;
        if (files.length <= 0) {
            return false;
        }
        
        var fr = new FileReader();
        
        fr.onload = function(e) { 
            try {
                result = JSON.parse(e.target.result);
                formatted = JSON.stringify(result);
                jsonErr.style.display = "none";
                jsonErr.innerHTML = "";
                console.log(formatted);
                localStorage.setItem(fileName, formatted);
            } catch (e) {
                jsonErr.innerHTML = e;
                jsonErr.style.display = "block";
            }
        }
        
        fr.readAsText(files.item(0));
        };

        document.getElementById('selectFiles').onchange =
        function updateFileName() {
            fileName = document.getElementById('selectFiles').value.replace(/.*[\/\\]/, '');
            if(fileName !== "") {
                document.getElementById('selectFileLabel').innerHTML = fileName;
            }
        }
    })
</script>

<div style="font-family: 'Poppins', sans-serif;" class="w-full h-full flex p-10 flex-col">
    <div class="flex-col [&>*]:text-white mb-3">
        <h1 class="font-light text-5xl pb-1">Star Systems</h1>
        <h2 class="font-extralight text-2xl pt-1">by Natalius</h2>
    </div>
    <hr class="my-3">
    <div class="flex-col [&>*]:text-white my-3">
        <p class="font-light text-lg">Get started by selecting a star system to visualise. If you would like to view a custom star system, you can also upload the file in the dialog below.</p>
    </div>
    <div class="bg-slate-600 py-5 px-7 rounded-md mt-3">
        <div class="flex-col [&>*]:text-white my-3">
            <h2 class="text-3xl my-5">Select Star System</h2>
            <!-- Stolen from https://jsfiddle.net/Ln37kqc0/ -->
            <div class="my-1"><select id="selectSystems" class="bg-slate-300 text-slate-800 px-3 py-2 rounded-sm border-slate-700 hover:underline cursor-pointer"></select></div>
            <div><Button class="mt-1 !mb-3" text="Open" id="open" /></div>
        </div>
        <div class="flex-col [&>*]:text-white mt-3">
            <h2 class="text-3xl my-5">Upload Star System</h2>
            <!-- Stolen from https://jsfiddle.net/Ln37kqc0/ -->
            <div class="my-1 flex-col flex"><label class="hover:underline w-fit text-slate-800 bg-slate-300 cursor-pointer px-3 py-2 rounded-sm"><span id="selectFileLabel">Submit File</span><input class="mt-3 mb-2 hidden" type="file" id="selectFiles"></label><span id="jsonErr" class="hidden text-red-500"></span></div>
            <div><Button class="mt-1 !mb-3" text="Upload" id="import" /></div>
        </div>
    </div>
</div>
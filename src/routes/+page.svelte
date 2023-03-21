<script>
    import Button from "../lib/Button.svelte"
    import { onMount } from "svelte";

    onMount(() => {
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
    <div class="flex-col [&>*]:text-white mb-2">
        <h1 class="font-light text-5xl pb-1">Star Systems</h1>
        <h2 class="font-extralight text-2xl pt-1">by Natalius</h2>
    </div>
    <hr class="my-3">
    <div class="flex-col [&>*]:text-white my-3">
        <p class="font-light text-lg">Get started by selecting a star system to visualise. If you would like to view a custom star system, you can also upload the file in the dialog below.</p>
    </div>
    <hr class="my-3 opacity-30">
    <div class="flex-col [&>*]:text-white my-3">
        <h2 class="text-3xl mb-5">Select Star System</h2>
        <!-- Stolen from https://jsfiddle.net/Ln37kqc0/ -->
        <div class="my-1 flex-col flex"><label class="hover:underline w-fit text-slate-800 bg-slate-300 cursor-pointer px-3 py-2 rounded-sm"><span id="selectFileLabel">Submit File</span><input class="mt-3 mb-2 hidden" type="file" id="selectFiles"></label><span id="jsonErr" class="hidden text-red-500"></span></div>
        <div><Button class="my-1" text="Upload" id="import" /></div>
    </div>
    <hr class="my-3 opacity-30">
    <div class="flex-col [&>*]:text-white mt-3">
        <h2 class="text-3xl mb-5">Upload Star System</h2>
        <!-- Stolen from https://jsfiddle.net/Ln37kqc0/ -->
        <div class="my-1 flex-col flex"><label class="hover:underline w-fit text-slate-800 bg-slate-300 cursor-pointer px-3 py-2 rounded-sm"><span id="selectFileLabel">Submit File</span><input class="mt-3 mb-2 hidden" type="file" id="selectFiles"></label><span id="jsonErr" class="hidden text-red-500"></span></div>
        <div><Button class="my-1" text="Upload" id="import" /></div>
    </div>
</div>
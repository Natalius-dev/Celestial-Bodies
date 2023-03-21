<script>
    import { onMount } from 'svelte';
    onMount(() => {
        const expandBtn = document.getElementById("expand-btn");
        const navbar = document.getElementById("navbar");
        const elements = document.getElementsByName("expand");
        expandBtn.onclick =
        function expandElements() {
            if(navbar.classList.contains("flex-row")) {
                navbar.classList.remove("flex-row");
                navbar.classList.add("flex-col");
            } else {
                navbar.classList.add("flex-row");
                navbar.classList.remove("flex-col");
            }
            for(let i = 0; i < elements.length; i++) {
                if(elements[i].classList.contains("!flex")) {
                    elements[i].classList.remove("!flex");
                } else {
                    elements[i].classList.add("!flex");
                }
            }
        }
        window.onresize = function resetElements() {
            if(getComputedStyle(expandBtn).display === "none") {
                navbar.classList.add("flex-row");
                navbar.classList.remove("flex-col");
            } else if (getComputedStyle(expandBtn).display !== "none" && getComputedStyle(elements[0]).display !== "none") {
                navbar.classList.remove("flex-row");
                navbar.classList.add("flex-col");
            }
        }
    });
</script>

<div id="navbar" style="font-family: 'Martian Mono', monospace;" class="flex flex-row font-thin [&>div]:hidden [&>div]:sm:flex [&>div]:justify-center [&>div]:duration-300 [&>div]:flex-grow [&>div]:text-white hover:[&>div]:text-gray-900 [&>div]:bg-slate-800 hover:[&>div]:bg-slate-500 hover:[&>div>a>span]:underline hover:[&>div>a>span]:text-black hover:[&>div>a>span]:duration-300 [&>div>a]:w-full [&>div>a]:h-full [&>div>a]:mx-5 [&>div>a]:my-4 [&>div>a]:text-center">
    <div id="expand-btn" class="!flex !justify-start sm:!hidden"><span class="mx-5 my-4"><i class="fa-solid fa-bars"></i></span></div>
    <div name="expand"><a href="#"><span>Home</span></a></div>
    <div name="expand"><a href="#"><span>About</span></a></div>
    <div name="expand"><a href="#"><span>FAQ</span></a></div>
    <div name="expand"><a href="#"><span>Contact</span></a></div>
</div>
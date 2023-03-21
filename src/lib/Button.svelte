<script>
    export let text;
    let clazz;
    export {clazz as class};
    export let id;
</script>

<style>
    .btn {box-shadow: 0 0.5rem 0 0 rgb(30, 41, 59); transition-duration: 200ms;}
    .btn:hover {box-shadow: 0 0.6rem 0 0 rgb(30, 41, 59); transform: translateY(-0.1rem);}
    .btn:active {box-shadow: 0 0rem 0 0 rgb(30, 41, 59); transform: translateY(0.5rem);}
</style>
<button id={id} class="bg-slate-300 px-5 py-2 rounded-sm overflow-hidden btn {clazz}"><span class="text-slate-800 font-medium select-none text-center">{text}</span></button>
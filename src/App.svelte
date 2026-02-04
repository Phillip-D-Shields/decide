<script>
  import FooterButtons from "./lib/FooterButtons.svelte";

  // --- STATE (using $state rune) ---
  let options = $state([
    {id: 1, text: 'Tacos'}, 
    {id: 2, text: 'Pizza'},
    {id: 3, text: 'Sushi'}
  ]);
  
  let isSpinning = $state(false);
  let nextId = $state(4);
  
  // Slot Machine Logic State
  let reelList = $state([]); 
  let translateY = $state(0); 
  let transitionDuration = $state(0); 
  
  const ITEM_HEIGHT = 80; 

  let validOptions = $derived(options.filter(o => o.text.trim() !== ''));
  let canSpin = $derived(validOptions.length >= 2 && !isSpinning);

  function addOption() {
    if (options.length < 10) {
      options.push({id: nextId++, text: ''});
    }
  }

  function removeOption(idToRemove) {
    if (options.length > 2) {
      options = options.filter(o => o.id !== idToRemove);
    }
  }

  async function spinSlot() {
    if (!canSpin) return;
    
    isSpinning = true;

    const repeatCount = 50; 
    reelList = Array(repeatCount).fill(validOptions).flat();

    transitionDuration = 0;
    translateY = 0;

    await new Promise(r => setTimeout(r, 50));

    const startIndexForWinners = reelList.length - validOptions.length;
    const randomOffset = Math.floor(Math.random() * validOptions.length);
    const winningIndex = startIndexForWinners + randomOffset;

    const finalPosition = winningIndex * ITEM_HEIGHT;

    transitionDuration = 3000; 
    translateY = -finalPosition; 

    setTimeout(() => {
      isSpinning = false;
    }, 3000);
  }
</script>

<main class="min-h-[100dvh] bg-base-200 flex flex-col items-center py-4 md:py-10 gap-4 md:gap-8 px-4 font-sans">
  
  <div class="text-center">
    <h1 class="text-3xl md:text-4xl font-black text-primary tracking-tight">DECIDE</h1>
  </div>

  <div class="relative p-2 md:p-4 bg-neutral rounded-3xl shadow-2xl border-4 border-base-content/10">
    
    <div class="relative w-64 h-20 bg-base-100 rounded-xl overflow-hidden border-2 border-primary/20 shadow-inner">
        
        <div class="absolute top-1/2 left-0 w-full h-0.5 bg-red-500/30 z-10 -translate-y-1/2"></div>
        <div class="absolute inset-0 shadow-[inset_0_0_20px_rgba(0,0,0,0.4)] pointer-events-none z-20 rounded-xl"></div>

        <div 
          class="flex flex-col items-center w-full"
          style="
            transform: translateY({translateY}px); 
            transition: transform {transitionDuration}ms cubic-bezier(0.2, 0.8, 0.2, 1);
          "
        >
          {#if reelList.length === 0}
             <div class="h-20 flex items-center justify-center text-xl font-bold uppercase text-base-content/50">
                Ready to Spin
             </div>
          {:else}
             {#each reelList as item, i}
                <div class="h-20 w-full flex items-center justify-center flex-shrink-0">
                  <span class="text-2xl font-black uppercase tracking-widest text-primary truncate px-4">
                    {item.text}
                  </span>
                </div>
             {/each}
          {/if}
        </div>

    </div>
    
    <div class="absolute -right-4 top-1/2 -translate-y-1/2 translate-x-full">
         <div class="w-4 h-12 bg-neutral rounded-r-lg"></div>
    </div>
  </div>

  <div class="w-full max-w-sm flex flex-col gap-4">
    <button 
      class="btn btn-primary btn-lg w-full shadow-lg shadow-primary/40 border-b-4 border-primary-focus active:border-b-0 active:translate-y-1 transition-all"
      disabled={!canSpin} 
      onclick={spinSlot}
    >
      {isSpinning ? 'Rolling...' : 'PULL LEVER'}
    </button>

    <div class="divider text-xs opacity-50 my-0">EDIT OPTIONS</div>

    <div class="flex flex-col gap-2 max-h-52 md:max-h-60 overflow-y-auto px-1">
      {#each options as option (option.id)}
        <div class="flex gap-2 animate-fade-in-up">
          <input 
            type="text" 
            bind:value={option.text} 
            placeholder="Option..." 
            class="input input-bordered w-full bg-base-100 shadow-sm focus:shadow-md transition-shadow" 
            disabled={isSpinning}
          />
          <button 
            class="btn btn-square btn-ghost text-error" 
            disabled={options.length <= 2 || isSpinning}
            onclick={() => removeOption(option.id)}
          >
            ✕
          </button>
        </div>
      {/each}
    </div>

    {#if options.length < 10}
      <button 
        class="btn btn-ghost btn-sm btn-block border-dashed border-2 border-base-300 opacity-60 hover:opacity-100" 
        onclick={addOption}
        disabled={isSpinning}
      >
        + Add Option
      </button>
    {/if}
    
  </div>

  <div class="w-full mt-auto pt-4 pb-2">
    <FooterButtons />
  </div>
</main>
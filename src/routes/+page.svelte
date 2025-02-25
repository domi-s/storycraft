<script>
    import { GoogleGenerativeAI } from "@google/generative-ai";

    const genAI = new GoogleGenerativeAI(import.meta.env.VITE_GOOGLE_API_KEY);
    const model = genAI.getGenerativeModel({ model: "gemini-1.5-flash" });


    const defaultPrompt = 'Generate a short example story on any topic, but make it interesting.';

    let isHappy = true;
    let isShort = true;
    let genre = 'any';
    let userInput = '';

    let prompt = '';
    let story = '';
    let title = '';
    let loading = false;

    let showDisclaimer = false;

    $: {
        let requirements = [];
        
        if (genre !== 'any') {
            requirements.push(`make it a ${genre} story`);
        }
        
        if (isShort) {
            requirements.push('keep it short and concise');
        }
        
        if (isHappy) {
            requirements.push('make sure it has a happy ending');
        }

        if(userInput.trim() !== '') {
            requirements.push(userInput[0].toLowerCase() + userInput.slice(1));
        }

        const requirementsText = requirements.length > 0 
            ? `What I would really like is for you to ${requirements.join(', ')}`
            : '';

        prompt = `Generate a story that's interesting and engaging. ${requirementsText}. Thanks!`;
    }

    async function generateStory() {
        if(loading) return;

        if(prompt.trim() === '')
            prompt = defaultPrompt;

        const aiPrompt = prompt + "\n\nPlease make sure that the first paragraph contains the title of the story!";

        loading = true;
        try {
            const result = await model.generateContent({
                contents: [{ role: 'user', parts: [{ text: aiPrompt }]}],
                generationConfig: {
                    maxOutputTokens: 500,
                },
            });
            const responseText = result.response.text();

            title = responseText.split('\n')[0];
            const titleOriginal = title;

            if(title[0] == '*')
                title = title.slice(2, -2);

            story = responseText.replace(titleOriginal, '').trim().replace(/\n/g, '<br>');
        } catch (error) {
            console.error('Error generating story:', error);
            story = 'An error occurred.';
        }
        loading = false;
    }
</script>

<div class="bg-amber-400">
    <div class="w-full max-w-screen-lg mx-auto p-4 flex flex-col items-center">
        <div class="flex flex-col items-center text-center gap-4 w-full">
            <div class="my-8">
                <h3 class="font-black text-amber-700">StoryCraft</h3>
                <h1 class="text-3xl font-black text-amber-900">Let's craft your story&hellip;</h1>
            </div>
            <div class="w-full">
                <div class="flex items-center gap-4 w-full">
                    <h3 class="font-bold flex-shrink-0 text-amber-900">Tweak your story</h3>
                    <div class="flex-grow h-[2px] bg-amber-900"></div>
                </div>
                <div class="w-full">
                    <div class="flex flex-wrap gap-4 my-4 justify-center">
                        <label class="flex items-center gap-2">
                            <input type="radio" name="genre" value="any" bind:group={genre}>
                            Any genre
                        </label>
                        <label class="flex items-center gap-2">
                            <input type="radio" name="genre" value="fantasy" bind:group={genre}>
                            Fantasy
                        </label>
                        <label class="flex items-center gap-2">
                            <input type="radio" name="genre" value="sci-fi" bind:group={genre}>
                            Sci-Fi
                        </label>
                        <label class="flex items-center gap-2">
                            <input type="radio" name="genre" value="mystery" bind:group={genre}>
                            Mystery
                        </label>
                        <label class="flex items-center gap-2">
                            <input type="radio" name="genre" value="romance" bind:group={genre}>
                            Romance
                        </label>
                        <label class="flex items-center gap-2">
                            <input type="radio" name="genre" value="children" bind:group={genre}>
                            Children
                        </label>
                    </div>
                    <div class="flex flex-wrap gap-4 my-4 justify-center">
                        <label class="flex items-center gap-2">
                            <input type="checkbox" bind:checked={isShort}>
                            Short Story
                        </label>
                        <label class="flex items-center gap-2">
                            <input type="checkbox" bind:checked={isHappy}>
                            Happy Ending
                        </label>
                    </div>
                    <div class="flex flex-col my-4 justify-center">
                        <label for="userInput" class="font-medium text-left">Is there anything else you would like to add to the story?</label>
                        <input type="text" placeholder="Tell me what you would like me to include here, e.g. the main character is a cat named Mittens." class="w-full py-2 outline-0" bind:value={userInput} name="userInput" id="userInput" />
                    </div>
                </div>
            </div>
            <div class="w-full">
                <div class="flex items-center gap-4 w-full">
                    <h3 class="font-bold flex-shrink-0 text-amber-900">Here's what I'm going to write about</h3>
                    <div class="flex-grow h-[2px] bg-amber-900"></div>
                </div>
                <div class="w-full">
                    <textarea readonly bind:value={prompt} placeholder={defaultPrompt} class="w-full py-2 outline-0 resize-none" rows="3"></textarea>
                    <button on:click={generateStory} disabled={loading} class="cursor-pointer border-amber-900 text-amber-900 font-bold border-2 border-solid py-2 px-4 rounded-lg">Generate Story</button>
                </div>
            </div>
        </div>
    </div>
</div>

<div>
    <div class="w-full max-w-screen-md mx-auto py-8 px-4 flex flex-col items-center">
        {#if loading}
            <p>Loading your story! Please hang tight&hellip;</p>
        {:else if story}
            <h3 class="font-bold text-xl text-center">{title}</h3>
            <p class="text-justify mt-8">{@html story}</p>
        {:else}
            <h3 class="font-bold text-xl text-center">No story generated yet.</h3>
            <p class="text-justify mt-8">Use the options above to generate a story.</p>
        {/if}
    </div>
</div>

<div>
    <div class="w-full max-w-screen-lg mx-auto py-8 px-4 flex flex-col items-start">
        <button class="font-medium text-xs cursor-pointer text-gray-500 underline decoration-dashed" on:click={() => showDisclaimer = !showDisclaimer}>Disclaimer</button>
        {#if showDisclaimer}
            <p class="text-sm mt-4">This is a fun project that uses the Google Generative AI model to generate stories based on user prompts. The stories are generated by the AI model and may not always make sense or be coherent. Depending on the prompt and the model, the stories may vary in quality and content. This project is for entertainment purposes only and should not be used for any serious or professional work. The stories generated are not guaranteed to be accurate, factual, correct, or suitable for any purpose. Use at your own risk.</p>
        {/if}
    </div>
</div>
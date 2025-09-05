<template>
  <!-- Main container with animated background -->
  <div class="h-full w-full bg-neutral-950 flex flex-col overflow-hidden relative">

    <!-- Background dots (subtle animation) -->
    <div class="absolute inset-0 z-0 opacity-20" style="background-image: radial-gradient(circle, rgba(255,255,255,0.05) 1px, transparent 1px); background-size: 20px 20px;"></div>


    <!-- Messages Area: Takes up all available space and scrolls -->
    <div class="flex-1 overflow-y-auto px-7 py-6 min-h-0 relative z-10" ref="messagesContainerRef">
      <div class="max-w-[758px] mx-auto">

        <!-- Welcome Screen Content: Shows ONLY if there are no messages -->
        <div v-if="messages.length === 0 && !isLoading"
             class="h-full flex flex-col justify-center items-center py-20 md:py-32"> <!-- Added vertical padding for better centering -->
          <div class="text-center max-w-2xl mx-auto px-6">
            <!-- Logo/Icon -->
            <div class="mb-8">
              <h1 class="text-4xl md:text-5xl font-extrabold text-white mb-2 leading-tight">Holy SHIT</h1>
              <p class="text-neutral-400 text-lg">AI is going to take over the fuckin' world.</p>
            </div>
            <div class="space-y-4">
              <p class="text-neutral-300 text-md">Start by asking me anything, for example:</p>
              <div class="flex flex-wrap justify-center gap-3">
                <button @click="startWithQuestion('What is quantum entanglement?')"
                        class="px-4 py-2 bg-neutral-800 text-neutral-200 rounded-lg hover:bg-neutral-700 transition-colors duration-200 text-sm shadow-md">
                  What is quantum entanglement?
                </button>
                <button @click="startWithQuestion('Explain React Hooks with an example.')"
                        class="px-4 py-2 bg-neutral-800 text-neutral-200 rounded-lg hover:bg-neutral-700 transition-colors duration-200 text-sm shadow-md">
                  Explain React Hooks with an example.
                </button>
                <button @click="startWithQuestion('Show me an example image.')"
                        class="px-4 py-2 bg-neutral-800 text-neutral-200 rounded-lg hover:bg-neutral-700 transition-colors duration-200 text-sm shadow-md">
                  Show me an example image.
                </button>
              </div>
            </div>
          </div>
        </div>

        <!-- Chat Messages: Appear as soon as the messages array is not empty -->
        <transition-group name="message" tag="div">
          <div v-for="message in messages" :key="message.id" class="mb-6">
            <!-- User Message -->
            <div v-if="message.type === 'user'" class="flex justify-end">
              <div class="bg-blue-600 bg-opacity-75 text-white px-5 py-3 rounded-2xl max-w-[758px] shadow-md break-words">
                {{ message.content }}
              </div>
            </div>

            <!-- AI Message -->
            <div v-else class="flex items-start">
              <div class="bg-neutral-800 bg-opacity-75 text-neutral-200 px-5 py-3 rounded-2xl max-w-[758px] shadow-md ai-message-content">
                <p class="text-sm text-neutral-400 font-semibold mb-1">Atlas</p>
                <!-- Render markdown here -->
                <div v-if="message.content.length > 0" v-html="renderMarkdown(message.content)"></div>
                <!-- Status message or typing indicator when AI is thinking/loading -->
                <div v-else>
                  <div v-if="isLoading && message.id == messages[messages.length-1].id" class="flex flex-col">
                      <p v-if="currentStatusMessage" class="text-neutral-400 text-sm italic animate-status-pulse mb-2 mt-[-4px]">
                        {{ currentStatusMessage }}
                      </p>
                      <div class="flex items-end space-x-1">
                          <div class="w-2.5 h-2.5 bg-neutral-500 rounded-full animate-typing-bounce" style="animation-delay: 0s"></div>
                          <div class="w-2.5 h-2.5 bg-neutral-500 rounded-full animate-typing-bounce" style="animation-delay: 0.1s"></div>
                          <div class="w-2.5 h-2.5 bg-neutral-500 rounded-full animate-typing-bounce" style="animation-delay: 0.2s"></div>
                      </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </transition-group>

        <!-- Error message display -->
        <div v-if="errorMessage" class="text-red-400 bg-red-900 bg-opacity-50 p-3 rounded-lg text-center mt-4 border border-red-700">
          {{ errorMessage }}
        </div>

      </div>
    </div>

    <!-- Input Area: Always visible at the bottom -->
    <div class="w-full flex justify-center px-4 pt-0 pb-4 relative z-10">
      <div class="max-w-[812px] w-full h-fit bg-neutral-900 rounded-2xl border border-neutral-600 focus-within:border-neutral-500 transition-colors duration-200 shadow-xl">
        <textarea
          v-model="inputText"
          ref="inputRef"
          @input="autoResize"
          @keydown.enter.prevent="handleSend"
          :placeholder="isLoading ? (currentStatusMessage || 'Working...') : 'Ask Atlas...'"
          :disabled="isLoading"
          class="bg-transparent text-neutral-100 resize-none px-5 py-4 outline-none max-h-[8rem] w-full placeholder-neutral-500 custom-scrollbar"
          rows="1"
        ></textarea>

        <div class="h-[32px] w-full text-neutral-200 flex justify-between items-center px-3 pb-2">
          <!-- Placeholder for future functionality like attachment -->
          <div class="cursor-pointer hover:text-white transition-colors duration-200 opacity-50" title="Coming soon">
            <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" fill="currentColor" viewBox="0 0 16 16">
              <path fill-rule="evenodd" d="M8 2a.5.5 0 0 1 .5.5v5h5a.5.5 0 0 1 0 1h-5v5a.5.5 0 0 1-1 0v-5h-5a.5.5 0 0 1 0-1h5v-5A.5.5 0 0 1 8 2"/>
            </svg>
          </div>
          <button
            class="transition-all duration-200 cursor-pointer"
            :class="inputText.trim().length > 0 && !isLoading ? 'opacity-100 text-blue-400 hover:text-blue-300' : 'opacity-30 cursor-not-allowed'"
            @click="handleSend"
            :disabled="!inputText.trim().length || isLoading"
            title="Send message"
          >
            <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" fill="currentColor" viewBox="0 0 16 16">
              <path d="M15.854.146a.5.5 0 0 1 .11.54l-5.819 14.547a.75.75 0 0 1-1.329.124l-3.178-4.995L.643 7.184a.75.75 0 0 1 .124-1.33L15.314.037a.5.5 0 0 1 .54.11ZM6.636 10.07l2.761 4.338L14.13 2.576zm6.787-8.201L1.591 6.602l4.339 2.76z"/>
            </svg>
          </button>
        </div>
      </div>
    </div>

  </div>
</template>

<script>
import axios from 'axios';
import { marked } from 'marked';
import katex from 'katex';
import 'katex/dist/katex.min.css'; // Import KaTeX CSS

export default {
  name: 'ChatComponent',

  props: {
    chatId: {
      required: true,
      type: String
    }
  },

  data() {
    return {
      messages: [],
      inputText: "",
      isLoading: false,
      errorMessage: null,
      backendUrl: "https://atlasai.up.railway.app/api",
      s3BaseUrl: "https://atlas07072025.s3.amazonaws.com/FitAI/Images/",
      currentStatusMessage: '' // New data property for status messages
    };
  },


  mounted() {
    this.setupMarkdown();
    if (this.$refs.inputRef) {
      this.$refs.inputRef.focus();
    }
    this.loadPreviousMessages();
  },

  watch: {
    messages: {
      handler() {
        this.scrollToBottom();
      },
      deep: true
    }
  },

  methods: {
    async loadPreviousMessages() {
      let id = 0;
      try {
        const response = await axios.post(`${this.backendUrl}/gethistory/`, {
          chatId: this.chatId,
        });

        const history = response.data.history;
        for (let i = 0; i < history.length; i++) {
          const message = history[i];

          // Push user message
          this.messages.push({
            id: ++id,
            type: 'user',
            content: message.prompt
          });

          // Process AI response to ensure it's a string
          let finalResponseContent = message.final_response;
          if (typeof finalResponseContent === 'object' && finalResponseContent !== null) {
            if (finalResponseContent.text) {
              finalResponseContent = finalResponseContent.text;
            } else if (finalResponseContent.code) {
              finalResponseContent = finalResponse.code;
            } else {
              // Fallback: stringify the entire object if no specific text/code property is found
              finalResponseContent = JSON.stringify(finalResponseContent, null, 2);
            }
          }

          // Push AI response
          this.messages.push({
            id: ++id,
            type: 'ai',
            content: finalResponseContent
          });
        }
      } catch (error) {
        console.error('Error loading chat history:', error);
        this.errorMessage = 'Failed to load chat history. Please refresh the page.';
      }
    },

    setupMarkdown() {
      // Create a custom renderer for marked
      const customRenderer = new marked.Renderer();

      // Override the 'code' method to handle standard code blocks
      customRenderer.code = (code, language) => {
        const langClass = language ? `language-${language}` : '';
        return `<pre><code class="${langClass}">${code.text}</code></pre>`;
      };

      // Apply the custom renderer
      marked.setOptions({
        breaks: true,
        gfm: true,
        renderer: customRenderer,
      });
    },

    renderMarkdown(markdownText) {
      if (!markdownText) return '';

      const safeMarkdownText = String(markdownText);
      let processedText = safeMarkdownText;

      const mathExpressions = []; // To store { id, content, displayMode }
      let mathCounter = 0;

      // Process block math (e.g., $$...$$)
      processedText = processedText.replace(/\$\$([\s\S]*?)\$\$/g, (match, content) => {
        const id = mathCounter++;
        mathExpressions.push({ id, content: content.trim(), displayMode: true });
        return `<!--MATH_BLOCK_PLACEHOLDER_${id}-->`;
      });

      // Process inline math (e.g., $...$)
      // Negative lookbehind/lookahead to avoid matching $$...$$
      processedText = processedText.replace(/(?<!\$)\$((?:(?!\$).)*?)\$(?!\$)/g, (match, content) => {
        const id = mathCounter++;
        mathExpressions.push({ id, content: content.trim(), displayMode: false });
        return `<!--MATH_INLINE_PLACEHOLDER_${id}-->`;
      });

      // Process the custom image format: ---filename.png--- and replace with placeholders
      const imagePlaceholders = {};
      let imageCounter = 0;
      processedText = processedText.replace(/---(.+?)---/g, (match, content) => {
        const imageExtensions = ['.png', '.jpg', '.jpeg', '.gif', '.webp'];
        const isImage = imageExtensions.some(ext => content.toLowerCase().endsWith(ext));

        if (isImage) {
          const imageUrl = `${this.s3BaseUrl}${content}`;
          const placeholder = `<!--IMAGE_PLACEHOLDER_${imageCounter}-->`;
          imagePlaceholders[placeholder] = `<p class="chat-image-container"><img src="${imageUrl}" alt="AI generated image: ${content}" class="ai-generated-image" loading="lazy" /></p>`;
          imageCounter++;
          return placeholder;
        }
        return match;
      });

      // Render the remaining markdown
      let html = marked.parse(processedText);

      // Replace KaTeX placeholders with rendered HTML
      for (const math of mathExpressions) {
        try {
          const renderedMath = katex.renderToString(math.content, {
            throwOnError: false, // Don't throw on error, show error message instead
            displayMode: math.displayMode,
          });
          // Replace the HTML comment placeholder with the rendered KaTeX HTML
          html = html.replace(
            new RegExp(`<!--MATH_(BLOCK|INLINE)_PLACEHOLDER_${math.id}-->`, 'g'),
            renderedMath
          );
        } catch (e) {
          console.error("KaTeX rendering error:", e);
          // Provide a user-friendly error message if KaTeX fails
          html = html.replace(
            new RegExp(`<!--MATH_(BLOCK|INLINE)_PLACEHOLDER_${math.id}-->`, 'g'),
            `<span class="text-red-500">KaTeX Error: ${e.message}</span>`
          );
        }
      }

      // Replace image placeholders with their full HTML
      for (const placeholder in imagePlaceholders) {
        html = html.replace(new RegExp(placeholder, 'g'), imagePlaceholders[placeholder]);
      }

      return html;
    },

    autoResize() {
      if (this.$refs.inputRef) {
        this.$refs.inputRef.style.height = 'auto';
        this.$refs.inputRef.style.height = Math.min(this.$refs.inputRef.scrollHeight, 128) + 'px';
      }
    },

    scrollToBottom() {
      this.$nextTick(() => {
        if (this.$refs.messagesContainerRef) {
          this.$refs.messagesContainerRef.scrollTop = this.$refs.messagesContainerRef.scrollHeight;
        }
      });
    },

    startWithQuestion(question) {
      this.inputText = question;
      this.$nextTick(() => {
        this.handleSend();
      });
    },

    async handleSend() {
      const trimmedInput = this.inputText.trim();
      if (!trimmedInput || this.isLoading) {
        return;
      }

      this.errorMessage = null;

      this.messages.push({
        id: Date.now(),
        type: 'user',
        content: trimmedInput
      });

      this.inputText = '';
      this.autoResize();

      this.isLoading = true;
      this.scrollToBottom();

      // Create AI message placeholder for streaming
      const aiMessageId = Date.now() + 1;
      this.messages.push({
        id: aiMessageId,
        type: 'ai',
        content: '' // Empty content to show status/loading
      });

      let aiContent = ''; // Accumulate AI response here

      try {
        const response = await fetch(`${this.backendUrl}/sendchat/`, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify({
            chatId: this.chatId,
            prompt: trimmedInput
          })
        });

        if (!response.ok) {
          // If the HTTP response is not OK, throw an error
          const errorText = await response.text(); // Get response body for more details
          throw new Error(`HTTP error! status: ${response.status}, message: ${errorText}`);
        }

        const reader = response.body.getReader();
        const decoder = new TextDecoder();
        let buffer = ''; // Buffer to handle incomplete JSON lines

        while (true) {
          const { done, value } = await reader.read();
          if (done) break;

          buffer += decoder.decode(value, { stream: true });
          const lines = buffer.split('\n');

          // Process all complete lines, keep the last incomplete one in buffer
          buffer = lines.pop();

          for (const line of lines) {
            if (line.startsWith('data: ')) {
              const data = line.slice(6);
              if (data === '[DONE]') {
                this.currentStatusMessage = ''; // Clear status once done
                break;
              }
              
              try {
                const parsed = JSON.parse(data);
                
                // Handle status message
                if (parsed.status) {
                  this.currentStatusMessage = parsed.status === 'calculating' ? 'Calculating...' : '';
                }

                // Handle streaming chunks
                if (parsed.chunk) {
                  aiContent += parsed.chunk;
                  // Update the AI message in real-time
                  const messageIndex = this.messages.findIndex(m => m.id === aiMessageId);
                  if (messageIndex !== -1) {
                    this.messages[messageIndex].content = aiContent;
                  }
                  this.scrollToBottom();
                } else if (parsed.final_response) {
                  // This is the final message, overwrite existing content
                  // This ensures any partial streaming content is replaced by the complete, final version.
                  const messageIndex = this.messages.findIndex(m => m.id === aiMessageId);
                  if (messageIndex !== -1) {
                    this.messages[messageIndex].content = parsed.final_response;
                  }
                  this.currentStatusMessage = ''; // Clear status for final response
                  this.scrollToBottom();
                }
              } catch (e) {
                console.warn('Error parsing stream chunk (may be expected for non-JSON lines):', e, 'Data:', data);
              }
            }
          }
        }

        // Final check: if no final_response was sent or it was empty, ensure the accumulated content is there
        const messageIndex = this.messages.findIndex(m => m.id === aiMessageId);
        if (messageIndex !== -1 && !this.messages[messageIndex].content) {
            this.messages[messageIndex].content = aiContent || "I'm sorry, I couldn't generate a response.";
        }

      } catch (error) {
        console.error('Error sending chat or streaming:', error);
        this.errorMessage = `Failed to connect to Atlas. Please try again. (${error.message})`;
        
        // Update the AI message with error content if it's still empty or partial
        const messageIndex = this.messages.findIndex(m => m.id === aiMessageId);
        if (messageIndex !== -1) {
          this.messages[messageIndex].content = aiContent || "Oops! It seems I encountered an error. Please try sending your message again.";
        }
      } finally {
        this.isLoading = false;
        this.currentStatusMessage = ''; // Clear status message on completion or error
        this.scrollToBottom();
      }
    }
  }
};
</script>

<style>
/* Basic custom scrollbar styles */
::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}
::-webkit-scrollbar-track {
  background: transparent;
}
::-webkit-scrollbar-thumb {
  background: #4a4a4a; /* Darker grey */
  border-radius: 4px;
}
::-webkit-scrollbar-thumb:hover {
  background: #6a6a6a; /* Lighter grey on hover */
}

/* For Firefox */
html {
  scrollbar-width: thin;
  scrollbar-color: #4a4a4a transparent;
}

/* Message rise-up animation for Vue transition-group */
.message-enter-active {
  transition: all 0.4s ease-out;
}
.message-enter-from {
  opacity: 0;
  transform: translateY(30px);
}
.message-enter-to {
  opacity: 1;
  transform: translateY(0);
}

/* Typing Dots Animation */
@keyframes typing-bounce {
  0%, 100% {
    transform: translateY(0);
    opacity: 0.5;
  }
  20% {
    transform: translateY(-8px) scale(1.1); /* Slightly higher bounce with subtle squash */
    opacity: 1;
  }
  40% {
    transform: translateY(0);
    opacity: 0.8;
  }
}
.animate-typing-bounce {
  animation: typing-bounce 1.2s infinite ease-in-out; /* Slower, smoother animation */
}

/* New Status Text Animation */
@keyframes status-pulse {
  0%, 100% {
    opacity: 0.7;
  }
  50% {
    opacity: 1;
  }
}
.animate-status-pulse {
  animation: status-pulse 1.8s infinite ease-in-out;
}


/* Enhanced Padding for Messages & Input */
/* User Message */
.bg-blue-600.bg-opacity-75.text-white.px-5.py-3 {
  padding-left: 1.25rem; /* px-5 */
  padding-right: 1.25rem; /* px-5 */
  padding-top: 0.75rem; /* py-3 */
  padding-bottom: 0.75rem; /* py-3 */
}
/* AI Message */
.bg-neutral-800.bg-opacity-75.text-neutral-200.px-5.py-3 {
  padding-left: 1.25rem; /* px-5 */
  padding-right: 1.25rem; /* px-5 */
  padding-top: 0.75rem; /* py-3 */
  padding-bottom: 0.75rem; /* py-3 */
}
/* Textarea */
.bg-transparent.text-neutral-100.resize-none.px-5.py-4 {
  padding-left: 1.25rem; /* px-5 */
  padding-right: 1.25rem; /* px-5 */
  padding-top: 1rem; /* py-4 */
  padding-bottom: 1rem; /* py-4 */
}

/* Markdown Styling for AI Messages */
.ai-message-content div {
  line-height: 1.6;
}

/* Headings */
.ai-message-content h1, .ai-message-content h2, .ai-message-content h3,
.ai-message-content h4, .ai-message-content h5, .ai-message-content h6 {
  @apply text-neutral-100 font-semibold mt-4 mb-2;
}
.ai-message-content h1 { @apply text-2xl; }
.ai-message-content h2 { @apply text-xl; }
.ai-message-content h3 { @apply text-lg; }
.ai-message-content h4 { @apply text-base; }

/* Paragraphs */
.ai-message-content p {
  @apply mb-3;
}

/* Links */
.ai-message-content a {
  @apply text-blue-400 hover:underline;
}

/* Lists - FIXED ALIGNMENT */
.ai-message-content ul, .ai-message-content ol {
  @apply list-outside mb-3 ml-4; /* Changed list-inside to list-outside, and pl-4 to ml-4 */
}
.ai-message-content ul {
  @apply list-disc;
}
.ai-message-content ol {
  @apply list-decimal;
}
.ai-message-content li {
  @apply mb-1;
}

/* Blockquotes */
.ai-message-content blockquote {
  @apply border-l-4 border-blue-500 pl-4 py-1 my-3 italic text-neutral-300;
}

/* Code Blocks */
.ai-message-content pre {
  @apply bg-neutral-900 text-neutral-200 p-3 rounded-md overflow-x-auto my-3;
  font-family: 'Fira Code', 'Cascadia Code', 'Consolas', 'Monaco', monospace;
}
.ai-message-content pre code {
  @apply block;
  padding: 0;
  background: none;
  font-size: 0.9em;
}

/* Inline Code */
.ai-message-content :not(pre) > code {
  @apply bg-neutral-700 text-neutral-100 px-1.5 py-0.5 rounded text-sm;
  font-family: 'Fira Code', 'Cascadia Code', 'Consolas', 'Monaco', monospace;
}

/* Tables (basic styling) */
.ai-message-content table {
  @apply w-full border-collapse my-4;
}
.ai-message-content th, .ai-message-content td {
  @apply border border-neutral-700 px-3 py-2;
}
.ai-message-content th {
  @apply bg-neutral-700 text-neutral-200 font-semibold;
}
.ai-message-content tr:nth-child(even) {
  @apply bg-neutral-800;
}

/* Horizontal Rule */
.ai-message-content hr {
  @apply border-t border-neutral-700 my-6;
}

/* Image Specific Styles */
.ai-message-content .chat-image-container {
  @apply my-4 flex justify-center;
}

.ai-message-content .ai-generated-image {
  @apply max-w-full h-auto rounded-lg shadow-lg border border-neutral-700;
  display: block;
  max-height: 400px;
  object-fit: contain;
}
</style>
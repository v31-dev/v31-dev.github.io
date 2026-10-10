<script setup lang="ts">
import { ref, onMounted } from "vue";
import axios from "axios";
import { Card, CardHeader, CardTitle, CardContent } from "@/components/ui/card";
import { ExternalLink } from "lucide-vue-next";
import { siGithub } from "simple-icons";

interface Repository {
  id: number;
  name: string;
  description: string | null;
  html_url: string;
  homepage: string | null;
  custom_properties?: Record<string, string | string[] | null>;
}

defineProps<{ msg: string }>();

const repos = ref<Repository[]>([]);
const loading = ref(true);
// Repos whose image URL failed to load fall back to the blank cover
const brokenImg = ref(new Set<number>());

const imgOf = (repo: Repository): string | null => {
  const url = repo.custom_properties?.showcase_img_url;
  if (typeof url !== "string" || !url.trim()) return null;
  return brokenImg.value.has(repo.id) ? null : url;
};

const fetchRepos = async () => {
  try {
    const { data } = await axios<Repository[]>(
      "https://api.github.com/orgs/v31-dev/repos",
    );
    repos.value = data.filter((r) => r.custom_properties?.showcase === "true");
  } catch (error) {
    console.error("Error fetching repos:", error);
  } finally {
    loading.value = false;
  }
};

onMounted(fetchRepos);
</script>

<template>
  <div class="w-full min-h-screen bg-background text-foreground p-8">
    <div class="max-w-6xl mx-auto">
      <h1 class="text-4xl font-bold mb-8 text-center">{{ msg }}</h1>

      <div v-if="loading" class="text-center py-12">
        <p class="text-muted-foreground">Loading repositories...</p>
      </div>

      <div
        v-else
        class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 auto-rows-fr gap-6"
      >
        <Card
          v-for="repo in repos"
          :key="repo.id"
          class="group h-full gap-4 overflow-hidden pt-0 bg-card border-border hover:border-primary transition-colors"
        >
          <!-- Fixed-height cover: same size with or without an image -->
          <div class="relative h-40 shrink-0 bg-muted">
            <template v-if="imgOf(repo)">
              <img
                :src="imgOf(repo)!"
                :alt="`${repo.name} preview`"
                loading="lazy"
                class="size-full object-cover object-center opacity-70 transition-opacity duration-300 group-hover:opacity-90"
                @error="brokenImg.add(repo.id)"
              />
              <div
                class="pointer-events-none absolute inset-0 bg-linear-to-t from-card to-transparent"
              ></div>
            </template>
          </div>

          <CardHeader>
            <div class="flex items-start justify-between gap-3">
              <CardTitle class="min-w-0 truncate text-xl text-foreground">{{
                repo.name
              }}</CardTitle>
              <div class="flex shrink-0 gap-2">
                <a
                  v-if="repo.html_url"
                  :href="repo.html_url"
                  target="_blank"
                  rel="noopener"
                  class="text-muted-foreground hover:text-foreground transition-colors"
                  title="View on GitHub"
                >
                  <div
                    class="w-5 h-5 [&>svg]:fill-current"
                    v-html="siGithub.svg"
                  ></div>
                </a>
                <a
                  v-if="repo.homepage"
                  :href="repo.homepage"
                  target="_blank"
                  rel="noopener"
                  class="text-muted-foreground hover:text-foreground transition-colors"
                  title="Visit homepage"
                >
                  <ExternalLink :size="20" />
                </a>
              </div>
            </div>
          </CardHeader>
          <CardContent>
            <p class="line-clamp-3 text-muted-foreground text-sm">
              {{ repo.description || "No description available" }}
            </p>
          </CardContent>
        </Card>
      </div>
    </div>
  </div>
</template>


const corsHeaders = {
  "Access-Control-Allow-Origin": "*",
  "Access-Control-Allow-Methods": "POST, OPTIONS",
  "Access-Control-Allow-Headers": "Content-Type",
};

export default {
  async fetch(request, env) {
    const url = new URL(request.url);

    if (request.method === "OPTIONS") {
      return new Response(null, { headers: corsHeaders });
    }

    if (url.pathname === "/") {
      return Response.json({
        success: true,
        message: "AI Content Lab API is running"
      }, { headers: corsHeaders });
    }

    if (url.pathname !== "/generate") {
      return Response.json(
        { success: false, error: "Endpoint not found" },
        { status: 404, headers: corsHeaders }
      );
    }

    if (request.method !== "POST") {
      return Response.json(
        { success: false, error: "Use POST method" },
        { status: 405, headers: corsHeaders }
      );
    }

    try {
      const data = await request.json();
      const prompt = data.prompt;

      if (!prompt || typeof prompt !== "string") {
        return Response.json(
          { success: false, error: "Please provide a prompt" },
          { status: 400, headers: corsHeaders }
        );
      }

      if (prompt.length > 6000) {
        return Response.json(
          { success: false, error: "Prompt is too long" },
          { status: 400, headers: corsHeaders }
        );
      }

      const result = await env.AI.run(
        "@cf/meta/llama-3.1-8b-instruct",
        {
          prompt: `You are AI Content Lab, a professional creative AI assistant. Follow the user's instructions carefully and produce a detailed, useful result.\n\n${prompt}`
        }
      );

      return Response.json(
        { success: true, result: result },
        { headers: corsHeaders }
      );

    } catch (error) {
      return Response.json(
        { success: false, error: "AI generation failed", details: error.message },
        { status: 500, headers: corsHeaders }
      );
    }
  }
};

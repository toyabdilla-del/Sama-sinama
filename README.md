import { GoogleGenAI } from '@google/genai';

export default async function handler(req, res) {
    if (req.method !== 'POST') {
        return res.status(405).json({ error: 'Method not allowed' });
    }

    try {
        const { query, mode } = req.body;
        if (!query) {
            return res.status(400).json({ error: 'Query text is required' });
        }

        // Initialize Gemini SDK with your secure server environment variable
        const ai = new GoogleGenAI({ apiKey: process.env.GEMINI_API_KEY });

        const sourceLang = mode === 'sinama-to-english' ? 'Sinama (Sama / Badjao dialect)' : 'English';
        const targetLang = mode === 'sinama-to-english' ? 'English' : 'Sinama (Sama / Badjao dialect)';

        const systemPrompt = `You are an expert linguist specializing in Austronesian languages, specifically the Sinama language spoken by the Sama/Badjao people of the Philippines, Malaysia, and Indonesia. 
Translate accurately between ${sourceLang} and ${targetLang}. 
Provide cultural nuances, specific dialect notes (e.g. Central Sinama, Southern Sinama), or grammatical insights if relevant.`;

        const userPrompt = `Translate the following text from ${sourceLang} to ${targetLang}: "${query}".
Return a JSON object with two fields:
1. "translation": The exact translated text in ${targetLang}.
2. "cultural_notes": Brief cultural context or usage notes if applicable, otherwise leave empty.`;

        const response = await ai.models.generateContent({
            model: 'gemini-2.5-flash',
            contents: userPrompt,
            config: {
                systemInstruction: systemPrompt,
                responseMimeType: "application/json",
                responseSchema: {
                    type: "OBJECT",
                    properties: {
                        translation: { type: "STRING" },
                        cultural_notes: { type: "STRING" }
                    },
                    required: ["translation"]
                }
            }
        });

        const data = JSON.parse(response.text());
        return res.status(200).json(data);

    } catch (error) {
        console.error("Backend Translation Error:", error);
        return res.status(500).json({ error: 'Failed to generate translation' });
    }
}

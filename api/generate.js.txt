import { createClient } from '@supabase/supabase-js';

const supabase = createClient(process.env.SUPABASE_URL, process.env.SUPABASE_ANON_KEY);
const ADMIN_PASSWORD = process.env.ADMIN_PASSWORD || 'your-secret-password';

export default async function handler(req, res) {
    if (req.method !== 'POST') {
        return res.status(405).json({ error: 'Method not allowed' });
    }

    const { password, count = 10 } = req.body;
    if (password !== ADMIN_PASSWORD) {
        return res.status(401).json({ error: 'Unauthorized' });
    }

    const tokens = [];
    for (let i = 0; i < count; i++) {
        const token = generateToken();
        tokens.push({ token, used: false });
    }

    const { error } = await supabase.from('tokens').insert(tokens);
    if (error) {
        console.error('Insert error:', error);
        return res.status(500).json({ error: error.message });
    }

    const baseUrl = process.env.BASE_URL || 'https://your-site.vercel.app';
    const links = tokens.map(t => `${baseUrl}?token=${t.token}`);
    return res.status(200).json({ links });
}

function generateToken() {
    return Math.random().toString(36).substring(2, 15) + Math.random().toString(36).substring(2, 15);
}
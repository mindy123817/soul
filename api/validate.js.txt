import { createClient } from '@supabase/supabase-js';

const supabase = createClient(process.env.SUPABASE_URL, process.env.SUPABASE_ANON_KEY);

export default async function handler(req, res) {
    // 允许跨域（方便本地调试）
    res.setHeader('Access-Control-Allow-Origin', '*');
    if (req.method === 'OPTIONS') {
        res.setHeader('Access-Control-Allow-Methods', 'GET, POST');
        return res.status(200).end();
    }

    if (req.method === 'GET') {
        const { token } = req.query;
        if (!token) {
            return res.status(400).json({ valid: false, error: 'Missing token' });
        }
        const { data, error } = await supabase
            .from('tokens')
            .select('used')
            .eq('token', token)
            .single();
        if (error || !data) {
            return res.status(200).json({ valid: false });
        }
        if (data.used) {
            return res.status(200).json({ valid: false });
        }
        return res.status(200).json({ valid: true });
    }

    if (req.method === 'POST') {
        const { token, markUsed } = req.body;
        if (!token || !markUsed) {
            return res.status(400).json({ error: 'Invalid request' });
        }
        const { error } = await supabase
            .from('tokens')
            .update({ used: true })
            .eq('token', token);
        if (error) {
            console.error('Update error:', error);
            return res.status(500).json({ error: 'Failed to mark used' });
        }
        return res.status(200).json({ success: true });
    }

    return res.status(405).json({ error: 'Method not allowed' });
}
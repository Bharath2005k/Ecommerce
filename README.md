import React, { useEffect, useState } from "react";
import { Bitcoin, Coins } from "lucide-react";
import { Select, SelectTrigger, SelectContent, SelectItem, SelectValue } from "@/components/ui/select";

const COINS = [
  {
    id: "bitcoin",
    name: "Bitcoin",
    symbol: "BTC",
    Icon: Bitcoin,
  },
  {
    id: "ethereum",
    name: "Ethereum",
    symbol: "ETH",
    Icon: Coins, // No specific Ethereum icon in allowed list, using Coins as fallback
  },
  {
    id: "tether",
    name: "Tether",
    symbol: "USDT",
    Icon: Coins,
  },
];

type ProbState = {
  up: number;
  down: number;
  trend: "up" | "down" | null;
};

function calculateProbability(current: number, prev: number | null): ProbState {
  if (prev === null) return { up: 50, down: 50, trend: null };
  if (current > prev) return { up: 75, down: 25, trend: "up" };
  if (current < prev) return { up: 30, down: 70, trend: "down" };
  return { up: 50, down: 50, trend: null };
}

const Index = () => {
  const [selectedCoin, setSelectedCoin] = useState<string>("bitcoin");
  const [price, setPrice] = useState<number | null>(null);
  const [prevPrice, setPrevPrice] = useState<number | null>(null);
  const [prob, setProb] = useState<ProbState>({
    up: 50,
    down: 50,
    trend: null,
  });
  const [loading, setLoading] = useState(true);

  const coinMeta = COINS.find((c) => c.id === selectedCoin)!;

  useEffect(() => {
    let mounted = true;
    const getPrice = async () => {
      try {
        setLoading(true);
        // fetch INR price
        const url = `https://api.coingecko.com/api/v3/simple/price?ids=${selectedCoin}&vs_currencies=inr`;
        const res = await fetch(url);
        const data = await res.json();
        if (mounted) {
          const newPrice = data[selectedCoin]?.inr;
          setPrevPrice(price !== null ? price : null);
          setPrice(newPrice);
          setLoading(false);
        }
      } catch {
        setLoading(false);
      }
    };

    getPrice();
    const interval = setInterval(getPrice, 60000);
    return () => {
      mounted = false;
      clearInterval(interval);
    };
    // eslint-disable-next-line
  }, [selectedCoin]);

  useEffect(() => {
    setProb(calculateProbability(price ?? 0, prevPrice));
  }, [price, prevPrice]);

  return (
    <div className="min-h-screen flex flex-col items-center justify-center bg-gradient-to-tr from-[#1a1f2c] via-[#232940] to-[#111317]">
      <div className="max-w-xl mx-auto px-6 py-16 rounded-3xl shadow-lg bg-white/5 border-t border-white/10 flex flex-col items-center gap-8">
        <div className="flex items-center gap-4">
          <span className="bg-yellow-400/15 rounded-full p-4 flex items-center justify-center">
            <coinMeta.Icon size={48} className="text-yellow-400" />
          </span>
          <h1 className="text-3xl md:text-4xl font-extrabold tracking-tight bg-gradient-to-r from-yellow-300 via-yellow-400 to-orange-400 bg-clip-text text-transparent">
            {coinMeta.name} Odds
          </h1>
        </div>
        <div className="w-full flex flex-col items-center gap-4">
          <Select
            value={selectedCoin}
            onValueChange={(value: string) => {
              setPrevPrice(null);
              setPrice(null);
              setSelectedCoin(value);
            }}
          >
            <SelectTrigger className="w-48 bg-black/40 text-lg text-white border border-gray-700 focus:ring-yellow-400">
              <SelectValue placeholder="Select a coin" />
            </SelectTrigger>
            <SelectContent className="z-20 bg-black/80 text-white">
              {COINS.map((c) => (
                <SelectItem key={c.id} value={c.id} className="flex items-center gap-2">
                  <span className="mr-2">
                    <c.Icon size={20} className="inline" />
                  </span>
                  <span>{c.name} ({c.symbol})</span>
                </SelectItem>
              ))}
            </SelectContent>
          </Select>
          <div className="flex flex-col items-center my-2">
            <h2 className="text-lg uppercase tracking-wider text-gray-400 mb-2">
              Current {coinMeta.name} Price
            </h2>
            <div className="flex items-end gap-2">
              <span className="text-5xl font-bold text-white drop-shadow">
                {loading ? (
                  <span className="animate-pulse text-gray-100">₹ ----</span>
                ) : price ? (
                  `₹${price.toLocaleString()}`
                ) : (
                  <span className="text-red-400">Error</span>
                )}
              </span>
              <span className="mb-1 text-gray-300 font-medium text-lg">INR</span>
            </div>
          </div>
        </div>
        <div className="w-full flex flex-col md:flex-row gap-8 items-center justify-center">
          <div className="flex flex-col items-center bg-black/20 rounded-xl p-6 min-w-[180px] shadow-lg">
            <svg className={`mb-2 ${prob.trend === "up" ? "text-green-400" : "text-gray-400"}`} width={36} height={36} viewBox="0 0 24 24" fill="none"><path d="M4 16L12 8L20 16" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"/></svg>
            <span className="text-2xl font-bold text-green-300">{prob.up}%</span>
            <span className="text-gray-300 mt-1">Chance to Rise</span>
          </div>
          <div className="flex flex-col items-center bg-black/20 rounded-xl p-6 min-w-[180px] shadow-lg">
            <svg className={`mb-2 ${prob.trend === "down" ? "text-red-400" : "text-gray-400"}`} width={36} height={36} viewBox="0 0 24 24" fill="none"><path d="M20 8L12 16L4 8" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round"/></svg>
            <span className="text-2xl font-bold text-red-300">{prob.down}%</span>
            <span className="text-gray-300 mt-1">Chance to Fall</span>
          </div>
        </div>
        <p className="text-xs text-gray-400 mt-6 max-w-lg text-center">
          Powered by real-time data from <a href="https://www.coingecko.com" target="_blank" rel="noopener noreferrer" className="underline text-yellow-300">Coingecko</a>. Probabilities are for demonstration and do not constitute financial advice.
        </p>
      </div>
    </div>
  );
};

export default Index;

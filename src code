//SRC Code for the puzzle game.

import { useState, useCallback, useEffect } from 'react';
import { useNavigate } from 'react-router-dom';
import { Button } from '@/components/ui/button';
import { ArrowLeft, RotateCcw } from 'lucide-react';
import { base44 } from '@/api/base44Client';
import FuturisticBackground from '@/components/game/FuturisticBackground';
import PuzzleBoard from '@/components/game/PuzzleBoard';
import GameHeader from '@/components/game/GameHeader';
import LevelComplete from '@/components/game/LevelComplete';

const GRID_SIZES = { easy: 3, medium: 4, hard: 5 };
const BASE_POINTS = { easy: 500, medium: 1000, hard: 2000 };
const TIME_BONUS_FACTOR = { easy: 10, medium: 15, hard: 25 };

export default function Game() {
  const navigate = useNavigate();
  const urlParams = new URLSearchParams(window.location.search);
  const difficulty = urlParams.get('difficulty') || 'easy';

  const [level, setLevel] = useState(1);
  const [totalScore, setTotalScore] = useState(0);
  const [startTime, setStartTime] = useState(Date.now());
  const [isRunning, setIsRunning] = useState(true);
  const [showComplete, setShowComplete] = useState(false);
  const [lastPoints, setLastPoints] = useState(0);
  const [puzzleKey, setPuzzleKey] = useState(0);

  const gridSize = GRID_SIZES[difficulty] || 3;

  useEffect(() => {
    setStartTime(Date.now());
    setIsRunning(true);
  }, [level, puzzleKey]);

  const calculatePoints = useCallback((timeSec) => {
    const base = BASE_POINTS[difficulty] || 500;
    const bonusFactor = TIME_BONUS_FACTOR[difficulty] || 10;
    const timeBonus = Math.max(0, Math.floor(bonusFactor * (120 - timeSec)));
    const levelBonus = level * 50;
    return base + timeBonus + levelBonus;
  }, [difficulty, level]);

  const handleSolve = useCallback(async (moves) => {
    setIsRunning(false);
    const timeSec = Math.floor((Date.now() - startTime) / 1000);
    const points = calculatePoints(timeSec);
    const newTotal = totalScore + points;

    setLastPoints(points);
    setTotalScore(newTotal);
    setShowComplete(true);

    // Save score
    const user = await base44.auth.me();
    await base44.entities.Score.create({
      player_name: user.full_name || user.email,
      player_email: user.email,
      points,
      level,
      difficulty,
      time_seconds: timeSec,
      total_score: newTotal,
    });
  }, [startTime, calculatePoints, totalScore, level, difficulty]);

  const handleNext = () => {
    setShowComplete(false);
    setLevel((l) => l + 1);
    setPuzzleKey((k) => k + 1);
  };

  const handleRestart = () => {
    setPuzzleKey((k) => k + 1);
    setStartTime(Date.now());
    setIsRunning(true);
  };

  return (
    <div className="min-h-screen relative flex flex-col items-center px-4 py-4">
      <FuturisticBackground level={level} />

      {/* Top bar */}
      <div className="w-full max-w-lg flex items-center justify-between mb-4">
        <Button
          variant="ghost"
          size="sm"
          onClick={() => navigate('/')}
          className="text-muted-foreground hover:text-foreground font-rajdhani rounded-xl"
        >
          <ArrowLeft className="w-4 h-4 mr-1" /> Exit
        </Button>
        <Button
          variant="ghost"
          size="sm"
          onClick={handleRestart}
          className="text-muted-foreground hover:text-foreground font-rajdhani rounded-xl"
        >
          <RotateCcw className="w-4 h-4 mr-1" /> Restart
        </Button>
      </div>

      <GameHeader
        level={level}
        difficulty={difficulty}
        totalScore={totalScore}
        startTime={startTime}
        isRunning={isRunning}
      />

      <div className="flex-1 flex items-center justify-center">
        <PuzzleBoard
          key={puzzleKey}
          size={gridSize}
          onSolve={handleSolve}
          level={level}
        />
      </div>

      {showComplete && (
        <LevelComplete
          level={level}
          points={lastPoints}
          totalScore={totalScore}
          onNext={handleNext}
        />
      )}
    </div>
  );
}

# Challenge

LovR is a fun game to play. Reach a certain level to see the hidden way.

ATTENTION: This Challenge does not use our flag format.

# Solution

We are presented with a game called match_3 created by the LÖVE game engine.

After googling a little, we see that we can obtain the source code of the game just by unzipping it. Unfortunately no flag hides in plain text.

However, after playing the game for a while, we notice a strange static on load

![static](/images/εικόνα.png)

(Note: some text seems to be hidden under the static but this is not very clear from the image)

So, we need to find what happens when we start the game. This is documented on script BeginGameState.lua and more specifically at function:

```Lua

function BeginGameState:render()
    
    
    self.board:render(self.myAlpha)

    
    love.graphics.setColor(95/255, 205/255, 228/255, 200/255)
    love.graphics.rectangle('fill', 0, self.levelLabelY - 8, VIRTUAL_WIDTH, 48)
    love.graphics.setColor(1, 1, 1, 1)
    love.graphics.setFont(gFonts['large'])
    love.graphics.printf('Level ' .. tostring(self.level),
        0, self.levelLabelY, VIRTUAL_WIDTH, 'center')
    love.graphics.draw(gTextures['dots'],0,30,0,0.19,0.2)
    if self.level == 10 then
        love.graphics.draw(gTextures['chars'],gFrames['char'][11],self.xxx,35,0,0.5,0.5)
        love.graphics.draw(gTextures['chars'],gFrames['char'][16],self.xxx+16,35,0,0.5,0.5)
        love.graphics.draw(gTextures['chars'],gFrames['char'][9],self.xxx+32,35,0,0.5,0.5)
        love.graphics.draw(gTextures['chars'],gFrames['char'][22],self.xxx+48,35,0,0.5,0.5)
        love.graphics.draw(gTextures['chars'],gFrames['char'][15],self.xxx+64,35,0,0.5,0.5)
        love.graphics.draw(gTextures['chars'],gFrames['char'][13],self.xxx+80,35,0,0.5,0.5)

        love.graphics.draw(gTextures['chars'],gFrames['char'][28],self.xxx+112,35,0,0.5,0.5)
        love.graphics.draw(gTextures['chars'],gFrames['char'][31],self.xxx+128,35,0,0.5,0.5)
        love.graphics.draw(gTextures['chars'],gFrames['char'][13],self.xxx+144,35,0,0.5,0.5)
        love.graphics.draw(gTextures['chars'],gFrames['char'][13],self.xxx+160,35,0,0.5,0.5)
        love.graphics.draw(gTextures['chars'],gFrames['char'][22],self.xxx+176,35,0,0.5,0.5)

        love.graphics.draw(gTextures['chars'],gFrames['char'][28],self.xxx,53,0,0.5,0.5)
        love.graphics.draw(gTextures['chars'],gFrames['char'][23],self.xxx+16,53,0,0.5,0.5)

        love.graphics.draw(gTextures['chars'],gFrames['char'][27],self.xxx+48,53,0,0.5,0.5)
        love.graphics.draw(gTextures['chars'],gFrames['char'][13],self.xxx+64,53,0,0.5,0.5)
        love.graphics.draw(gTextures['chars'],gFrames['char'][13],self.xxx+80,53,0,0.5,0.5)

        love.graphics.draw(gTextures['chars'],gFrames['char'][14],self.xxx+112,53,0,0.5,0.5)
        love.graphics.draw(gTextures['chars'],gFrames['char'][20],self.xxx+128,53,0,0.5,0.5)
        love.graphics.draw(gTextures['chars'],gFrames['char'][9],self.xxx+144,53,0,0.5,0.5)
        love.graphics.draw(gTextures['chars'],gFrames['char'][15],self.xxx+160,53,0,0.5,0.5)
        
    end
    love.graphics.draw(gTextures['dots2'],0,180,0,0.19,0.2)
    love.graphics.draw(gTextures['chars'],gFrames['char'][14],self.xx,185,0,0.75,0.75)
    love.graphics.draw(gTextures['chars'],gFrames['char'][20],self.xx+16,185,0,0.75,0.75)
    love.graphics.draw(gTextures['chars'],gFrames['char'][4],self.xx+32,185,0,0.75,0.75)
    love.graphics.draw(gTextures['chars'],gFrames['char'][15],self.xx+48,185,0,0.75,0.75)
    love.graphics.draw(gTextures['slay'],self.xx+64,205,0,0.75,0.75)
    love.graphics.draw(gTextures['chars'],gFrames['char'][28],self.xx+80,185,0,0.75,0.75)
    love.graphics.draw(gTextures['chars'],gFrames['char'][31],self.xx+96,185,0,0.75,0.75)
    love.graphics.draw(gTextures['chars'],gFrames['char'][3],self.xx+112,185,0,0.75,0.75)
    love.graphics.draw(gTextures['chars'],gFrames['char'][3],self.xx+128,185,0,0.75,0.75)
    love.graphics.draw(gTextures['chars'],gFrames['char'][22],self.xx+144,185,0,0.75,0.75)
  
    love.graphics.setColor(1, 1, 1, self.transitionAlpha)
    love.graphics.rectangle('fill', 0, 0, VIRTUAL_WIDTH, VIRTUAL_HEIGHT)
    
end

```

If we delete the lines:
```Lua
love.graphics.draw(gTextures['dots'],0,30,0,0.19,0.2)
love.graphics.draw(gTextures['dots2'],0,180,0,0.19,0.2)
```

We can patch the level check (if self.level == 10) to if self.level == 1, so that another line of text will be generated:

We then recompile the game, we view the screen:

![flag](/images/flag_lovr.png)

So the flag is FL4G_TW33N!!

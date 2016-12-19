if GetObjectName(myHero) ~= "Vayne" then return end

local ver = "0.06"

function AutoUpdate(data)
    if tonumber(data) > tonumber(ver) then
        print("New version found! " .. data)
        print("Downloading update, please wait...")
        DownloadFileAsync("https://raw.githubusercontent.com/Toshibiotro/stuff/master/EternalVayne.lua", SCRIPT_PATH .. "EternalVayne.lua", function() print("Update Complete, please 2x F6!") return end)
    end
end
GetWebResultAsync("https://raw.githubusercontent.com/Toshibiotro/stuff/master/EternalVayne.version", AutoUpdate)

require ("OpenPredict")
require ("MapPositionGOS")
require ("ChallengerCommon")


local VayneMenu = Menu("Vayne", "Vayne")
VayneMenu:SubMenu("Combo", "Combo")
VayneMenu.Combo:Boolean("CQ", "Use Q", true)
VayneMenu.Combo:SubMenu("EO", "E Options")
VayneMenu.Combo.EO:Boolean("EC", "Use E To Peel", true)
VayneMenu.Combo.EO:Slider("ECC", "Push Distance",125,0,550,5)
VayneMenu.Combo.EO:Boolean("ES", "E To Stun", true)
VayneMenu.Combo:Boolean("CR", "Use R", true)
VayneMenu.Combo:Slider("CMM", "Min Mana To Combo",20,0,100,1)
VayneMenu.Combo:Boolean("BORK", "Use BORK", true)
VayneMenu.Combo:Boolean("Bilge", "Use Bilge", true)
VayneMenu.Combo:Boolean("RotSec", "ZZRot Condemn", true)
VayneMenu.Combo:Boolean("CF", "Condemn Flash Combo", true)

VayneMenu:SubMenu("Harass", "Harass")
VayneMenu.Harass:Boolean("HQ", "Use Q", true)
VayneMenu.Harass:SubMenu("HEO", "E Options")
VayneMenu.Harass.HEO:Boolean("HEC", "Use E To Peel", true)
VayneMenu.Harass.HEO:Slider("HECC", "Push Distance",125,0,550,5)
VayneMenu.Harass.HEO:Boolean("HES", "Use E To Stun", true)
VayneMenu.Harass:Slider("HMM", "Min Mana To Harass",20,0,100,1)

VayneMenu:SubMenu("LaneClear", "LaneClear")
VayneMenu.LaneClear:Boolean("LCQ", "Use Q", true)
VayneMenu.LaneClear:Slider("LCMM", "Min Mana To LaneClear",20,0,100,1)

VayneMenu:SubMenu("JungleClear", "JungleClear")
VayneMenu.JungleClear:Boolean("JCQ", "Use Q", true)
VayneMenu.JungleClear:Boolean("JCE", "Use E", true)
VayneMenu.JungleClear:Slider("JCMM", "Min Mana To JungleClear",20,0,100,1)

VayneMenu:SubMenu("KillSteal", "KillSteal")
VayneMenu.KillSteal:Boolean("KSQ", "Use Q", true)
VayneMenu.KillSteal:Boolean("KSW", "Use W", true)

VayneMenu:SubMenu("Misc", "Misc")
VayneMenu.Misc:DropDown("AutoLevel", "AutoLevel", 1, {"Off", "QEW", "QWE", "WQE", "WEQ", "EQW", "EWQ", "Toshi Special"})
VayneMenu.Misc:Boolean("AC", "Auto Condemn", true)
VayneMenu.Misc:Boolean("ACA", "Anivia Wall Condemn", true)
VayneMenu.Misc:Boolean("ACT", "Trundle Pillar Condemn", true)
VayneMenu.Misc:Boolean("QSS", "Use QSS", true)
VayneMenu.Misc:Slider("QSSC", "HP To QSS", 90,0,100,1)
VayneMenu.Misc:Boolean("AI", "Auto Ignite", true)
VayneMenu.Misc:Boolean("AR", "Auto R", true)
VayneMenu.Misc:Slider("ARC", "Min Enemies To Auto R",3,1,6,1)
VayneMenu.Misc:Boolean("DAAS", "Don't AA While Invis", true)
VayneMenu.Misc:Boolean("QAC", "Q Animation Cancel", true)

VayneMenu:SubMenu("GapClose", "GapClose")
VayneMenu.GapClose:Boolean("GCQ", "Use Q", true)
VayneMenu.GapClose:Boolean("GCR", "Use R", false)

VayneMenu:SubMenu("AntiGapCloser", "AntiGapCloser")

VayneMenu:SubMenu("Interrupter", "Interrupter")

VayneMenu:SubMenu("Draw", "Drawings")
VayneMenu.Draw:Boolean("DAA", "Draw AA Range", true)
VayneMenu.Draw:Boolean("DQ", "Draw Q Range", true)
VayneMenu.Draw:Boolean("DE", "Draw E Range", true)
VayneMenu.Draw:Boolean("DWD", "Draw W Damage", true)

VayneMenu:SubMenu("SkinChanger", "SkinChanger")

local skinMeta = {["Vayne"] = {"Classic", "Vindicator", "Aristocrat", "DragonSlayer", "Heartseeker", "SKT T1", "Arclight", "DragonSlayer Green", "DragonSlayer Red", "DragonSlayer Blue", "SoulStealer"}}
VayneMenu.SkinChanger:DropDown('skin', myHero.charName.. " Skins", 1, skinMeta[myHero.charName], HeroSkinChanger, true)
VayneMenu.SkinChanger.skin.callback = function(model) HeroSkinChanger(myHero, model - 1) print(skinMeta[myHero.charName][model] .." ".. myHero.charName .. " Loaded!") end	

local target = GetCurrentTarget()
function QDmg(unit) return CalcDamage(myHero, unit, myHero.totalDamage + (myHero.totalDamage * (0.25 + 0.05 * GetCastLevel(myHero, _W))), 0) end
function WDmg(unit) return CalcDamage(myHero, unit, AADmg(unit), 0) + (unit.maxHealth * (0.045 + 0.015 * GetCastLevel(myHero, _W))) end
function AADmg(unit) return CalcDamage(myHero, unit, myHero.totalDamage, 0) end
function EDmg(unit) return CalcDamage(myHero, unit, (10 + 35 * GetCastLevel(myHero, _E)) + (myHero.totalDamage * 0.5)) end
function WStacks(unit) return GetBuffData(unit, "VayneSilveredDebuff").Count end
local Move = {delay = 0.5, speed = math.huge, width = 50, range = math.huge}
local CCType = {[5] = "Stun", [7] = "Silence", [8] = "Taunt", [9] = "Polymorph", [11] = "Snare", [21] = "Fear", [22] = "Charm", [24] = "Suppression"}
local QSS = nil
local MercSkimm = nil
local AARange = GetRange(myHero) + GetHitBox(myHero)
local ERange = GetCastRange(myHero, _E) + GetHitBox(myHero)
local QRange = GetCastRange(myHero, _Q)
local CPos = nil
local ZSpot = nil
local ZZRot = nil
local WallC = nil
local WallT = nil
local EStats = {delay = 0.25, range = ERange, radius = 1, speed = 2000}
local Meh = nil
local Meh2 = nil
local SMeh = nil
local SMeh2 = nil
local FFPos = nil
local FFPos2 = nil
local FFPos3 = nil
local CFPos = nil
local CFPos2 = nil
local CFPos3 = nil
local flash = (GetCastName(GetMyHero(),SUMMONER_1):lower():find("summonerflash") and SUMMONER_1 or (GetCastName(GetMyHero(),SUMMONER_2):lower():find("summonerflash") and SUMMONER_2 or nil))
local Pink = nil

function Mode()
    if _G.IOW_Loaded and IOW:Mode() then
        return IOW:Mode()
        elseif _G.PW_Loaded and PW:Mode() then
        return PW:Mode()
        elseif _G.DAC_Loaded and DAC:Mode() then
        return DAC:Mode()
        elseif _G.AutoCarry_Loaded and DACR:Mode() then
        return DACR:Mode()
        elseif _G.SLW_Loaded and SLW:Mode() then
        return SLW:Mode()
    end
end

function ResetAA()
    if _G.IOW_Loaded then
        return IOW:ResetAA()
        elseif _G.PW_Loaded then
        return PW:ResetAA()
        elseif _G.DAC_Loaded then
        return DAC:ResetAA()
        elseif _G.AutoCarry_Loaded then
        return DACR:ResetAA()
        elseif _G.SLW_Loaded then
        return SLW:ResetAA()
    end
end

function BlockAttack(boolean)
	if _G.IOW_Loaded then
		return IOW.attacksEnabled == (boolean)
		elseif _G.PW_Loaded then
        return PW.attacksEnabled == (boolean)
        elseif _G.DAC_Loaded then
        return DAC.attacksEnabled == (boolean)
        elseif _G.AutoCarry_Loaded then
        return DACR.attacksEnabled == (boolean)
        elseif _G.SLW_Loaded then
        return SLW.attacksEnabled == (boolean)
    end
end

OnTick(function()
	
	target = GetCurrentTarget()
	local IDamage = (50 + (20 * GetLevel(myHero)))
	local BORK = GetItemSlot(myHero, 3153)
	local Bilge = GetItemSlot(myHero, 3144)
	QSS = GetItemSlot(myHero, 3140)
	MercSkimm = GetItemSlot(myHero, 3139)
	local movePos = GetPrediction(target,Move).castPos
	CPos = target.pos + (target.pos - myHero.pos):normalized() * 430
	ZSpot = target.pos + (target.pos - myHero.pos):normalized() * 100
	ZZRot = GetItemSlot(myHero, 3512)
	local Invis = GotBuff(myHero, "vaynetumblefade")
	
	FFPos = target.pos + (target.pos - myHero.pos):normalized() * 425
	FFPos2 = target.pos + (target.pos - myHero.pos):perpendicular():normalized() * 425
	FFPos3 = target.pos + (target.pos - myHero.pos):perpendicular2():normalized() * 425
	FFPos4 = target.pos + (target.pos - myHero.pos):normalized() * -425
	
	CFPos = target.pos + (target.pos - FFPos):normalized() * 440
	CFPos2 = target.pos + (target.pos - FFPos):normalized():perpendicular() * 440
	CFPos3 = target.pos + (target.pos - FFPos):normalized():perpendicular2() * 440
	CFPos4 = target.pos + (target.pos - FFPos):normalized() * -440
	
	SMehPus = GetPrediction(target, EStats)
	SMehPos = Vector(SMehPus)
	SMeh = SMehPos + (SMehPos - myHero.pos):normalized() * 420
	SMeh2 = SMehPos + (SMehPos - myHero.pos):normalized() * 180

	
	if Invis > 0 and Pink ~= nil and GetDistance(myHero, Pink) > 1000 and VayneMenu.Misc.DAAS:Value() and EnemiesAround(myHero, 800) > 0 and GetDistance(myHero, target) < 300 then BlockAttack(true)
		elseif Pink == nil and VayneMenu.Misc.DAAS:Value() and EnemiesAround(myHero, 800) > 0 and GetDistance(myHero, target) < 300 then BlockAttack(true)
	end
	
	if Invis < 1 then BlockAttack(false) end
	
	--AutoLevel
	if VayneMenu.Misc.AutoLevel:Value() == 2 then
		spellorder = {_Q, _E, _W, _Q, _Q, _R, _Q, _E, _Q, _E, _R, _E, _E, _W, _W, _R, _W, _W}
		if GetLevelPoints(myHero) > 0 then
			LevelSpell(spellorder[GetLevel(myHero) + 1 - GetLevelPoints(myHero)])
		end
	end

	if VayneMenu.Misc.AutoLevel:Value() == 3 then
		spellorder = {_Q, _W, _E, _Q, _Q, _R, _Q, _W, _Q, _W, _R, _W, _W, _E, _E, _R, _E, _E}
		if GetLevelPoints(myHero) > 0 then
			LevelSpell(spellorder[GetLevel(myHero) + 1 - GetLevelPoints(myHero)])
		end
	end
	
	if VayneMenu.Misc.AutoLevel:Value() == 4 then
		spellorder = {_W, _Q, _E, _W, _W, _R, _W, _Q, _W, _Q, _R, _Q, _Q, _E, _E, _R, _E, _E}
		if GetLevelPoints(myHero) > 0 then
			LevelSpell(spellorder[GetLevel(myHero) + 1 - GetLevelPoints(myHero)])
		end
	end
	
	if VayneMenu.Misc.AutoLevel:Value() == 5 then
		spellorder = {_W, _E, _Q, _W, _W, _R, _W, _E, _W, _E, _R, _E, _E, _Q, _Q, _R, _Q, _Q}
		if GetLevelPoints(myHero) > 0 then
			LevelSpell(spellorder[GetLevel(myHero) + 1 - GetLevelPoints(myHero)])
		end
	end
	
	if VayneMenu.Misc.AutoLevel:Value() == 6 then
		spellorder = {_E, _Q, _W, _E, _E, _R, _E, _Q, _E, _Q, _R, _Q, _Q, _W, _W, _R, _W, _W}
		if GetLevelPoints(myHero) > 0 then
			LevelSpell(spellorder[GetLevel(myHero) + 1 - GetLevelPoints(myHero)])
		end
	end
	
	if VayneMenu.Misc.AutoLevel:Value() == 7 then
		spellorder = {_E, _W, _Q, _E, _E, _R, _E, _W, _E, _W, _R, _W, _W, _Q, _Q, _R, _Q, _Q}
		if GetLevelPoints(myHero) > 0 then
			LevelSpell(spellorder[GetLevel(myHero) + 1 - GetLevelPoints(myHero)])
		end
	end
	
	if VayneMenu.Misc.AutoLevel:Value() == 8 then
		spellorder = {_Q, _W, _E, _Q, _Q, _R, _W, _W, _W, _W, _R, _Q, _Q, _E, _E, _R, _E, _E}
		if GetLevelPoints(myHero) > 0 then
			LevelSpell(spellorder[GetLevel(myHero) + 1 - GetLevelPoints(myHero)])
		end
	end		
	
	--Combo
	if Mode() == "Combo" then
		
		if VayneMenu.Combo.EO.EC:Value() and Ready(_E) and ValidTarget(target, ERange) then
			if GetPercentMP(myHero) >= VayneMenu.Combo.CMM:Value() and GetDistance(myHero, target) <= VayneMenu.Combo.EO.ECC:Value() then
				CastTargetSpell(target, _E)
			end
		end
		
		if VayneMenu.Combo.EO.ES:Value() and Ready(_E) and ValidTarget(target, ERange) then
			if GetPercentMP(myHero) >= VayneMenu.Combo.CMM:Value() and MapPosition:inWall(SMeh) or MapPosition:inWall(SMeh2) then
				blahblah = false
				blahblah2 = false
				blahblah3 = false
				blahblah4 = false
				CastTargetSpell(target, _E)
			end
		end		

		if VayneMenu.Combo.CR:Value() and Ready(_R) and ValidTarget(target, 750) then
			if GetPercentMP(myHero) >= VayneMenu.Combo.CMM:Value() then
				if GetPercentHP(myHero) >= GetPercentHP(target) and GetPercentHP(target) >= 30 then
					CastSpell(_R)
				end
			end	
		end		
		
		if VayneMenu.Combo.BORK:Value() and Ready(BORK) and ValidTarget(target, 550) then
			if GetPercentHP(myHero) <= 90 and GetDistance(movePos) < GetDistance(target) then
				CastTargetSpell(target,

-- // load

repeat task.wait() until library and library.has_init

-- // variables
-- globals
local color3_new  = Color3.new
local color3_rgb  = Color3.fromRGB
local cframe_new  = CFrame.new
local vector3_new = Vector3.new
local vector2_new = Vector2.new
local math_clamp  = math.clamp
local math_floor  = math.floor
local math_max    = math.max
local math_abs    = math.abs
local math_tan    = math.tan
local math_rad    = math.rad
local math_sin    = math.sin
local math_cos    = math.cos
local math_atan2  = math.atan2

-- locals
local camera = workspace.CurrentCamera
local utility = library.utility

-- // functions 
local esp = {
    fps = 60,
    lastrender = 0,
    autostep = true,
    classes = {},
    objects = {},
    rayignore = {}
}

for i,v in next, ({...})[1] or {} do
    esp[i] = v
end

function esp.unload()
    for i,v in next, esp.objects do
        v:remove()
    end
    esp.nuke(esp)
    getgenv().esp = nil
end

function esp.define(class, properties, constructor)
    local object = {
        _class = class,
        _properties = properties,
        _constructor = constructor
    }

    esp.classes[class] = object
    return object
end

function esp.create(class, properties)
    local class = esp.classes[class]
    local object = class._constructor(utility.table.deepcopy(class._properties))

    for i,v in next, properties or {} do
        object[i] = v
    end

    return object
end

function esp.instance(class, properties)
    local instance = Instance.new(class)
    for i,v in next, properties or {} do
        instance[i] = v
    end
    return instance
end

function esp.get_box_size(screen_position, cframe, size)
    local x = utility.camera.cframetoviewport(cframe * cframe_new(size.X, 0, 0)); x = vector2_new(x.X, x.Y)
    local y = utility.camera.cframetoviewport(cframe * cframe_new(0, size.Y , 0)); y = vector2_new(y.X, y.Y)
    local z = utility.camera.cframetoviewport(cframe * cframe_new(0, 0, size.Z)); z = vector2_new(z.X, z.Y)
    local SizeX = math_max(math_abs(screen_position.X - x.X), math_abs(screen_position.X - z.X))
    local SizeY = math_max(math_abs(screen_position.Y - y.Y), math_abs(screen_position.Y - x.Y))
    return vector2_new(math_floor(math_clamp(SizeX,3,9999)), math_floor(math_clamp(SizeY,6, 9999)))
end

function esp:remove()
    if self._group ~= nil then
        self._group._objects[self] = nil
    end

    for i,v in next, self._components._drawings or {} do
        v:Remove()
    end

    for i,v in next, self._components._text_drawings or {} do
        v:Remove()
    end

    for i,v in next, self._connections or {} do
        v:Disconnect()
    end

    if table.find(esp.objects, self) then
        esp.objects[self] = nil
    end
    table.clear(self)
end

function esp:set_visible(bool)
    if not bool and self._visible then
        self:_set_all('Visible', false)
    elseif bool and not self._visible then
        self:_set_all('Visible', true)
    end
    self._visible = bool
end

-- // esp

local esp_object = {
    __index = function(self, idx)
        if rawget(self, idx) ~= nil then
            return rawget(self, idx)
        elseif esp[idx] ~= nil then
            return esp[idx]
        elseif rawget(self, '_properties') then
            return rawget(self, '_properties')[idx]
        end
    end,
    __newindex = function(self, idx, val)
        if self._objects ~= nil then
            for _, object in next, self._objects do
                object[idx] = val
            end
        end
        if self._properties ~= nil then
            self._properties[idx] = val
        end
    end
}

esp.define('group', {
    _enabled = false,
    _properties = {
        maxdistance = 8000
    },
    _objects = {},
}, function(object)
    return setmetatable(object, esp_object)
end)

esp.define('player', {
    _cframe = CFrame.new(0, 0, 0),
    _offset = CFrame.new(0, -1.2, 0),
    _size = Vector3.new(4, 6.5, 1.5),
    _visible = false,
    _connections = {},
    _properties = {
        enabled = true,
        dynamic = false,
        health = 0,
        maxhealth = 0,

        vischeck = false,
        vischeck_color = color3_rgb(100, 255, 100),
        notvischeck = false,
        notvischeck_color = color3_rgb(255,100,100),

        friendcheck = false,
        friend_color = color3_rgb(100, 115, 255),

        targetable_check = false,
        targetable_color = color3_new(0.75, 0.25, 1),

        targetcheck = false,
        target_color = color3_new(1,0,1),

        box = false,
        box_color = color3_new(1,1,1),

        healthbar = false,
        health_color_1 = color3_new(1,0,0),
        health_color_2 = color3_new(0,1,0),

        arrow = false,
        arrow_radius = 300,
        arrow_size = 15,
        arrow_color = color3_new(1,1,1),
        arrow_pulse = false,
        arrow_pulse_distance = 0,
        arrow_pulse_color = color3_new(1,1,1),
        arrow_options = {},

        text = false,
        text_color = color3_new(1,1,1),
        text_layout = {},

        blockchams = false,
        blockchams_color = BrickColor.new(color3_new(1,1,1)),
        blockchams_opacity = 0,

        chams = false,
        chams_occluded = false,
        chams_opacity_1 = 0,
        chams_opacity_2 = 0,
        chams_color_1 = color3_new(1,1,1),
        chams_color_2 = color3_new(1,1,1),
    },
    _set_all = function(self, property, value)
        for _, drawing in next, self._components._drawings do drawing[property] = value end
        for _, drawing in next, self._components._text_drawings do drawing[property] = value end
    end,
    _check = function(self)

        if not self.enabled or not self._group._enabled or 0 >= self.health then
            return false
        end

        local cframe = (self._model ~= nil and self._model.PrimaryPart ~= nil) and self._model.PrimaryPart.CFrame or self._cframe
        if cframe == nil or (cframe.p - camera.CFrame.p).magnitude / 3 > self._group.maxdistance then
            return false
        end

        local health_factor = self.health / self.maxhealth
        local screen_position, screen_visible = utility.camera.cframetoviewport(cframe * self._offset)

        return screen_visible, {
            cframe = cframe,
            screen_position = utility.vector2.floor(screen_position),
            screen_visible = screen_visible,
            distance = screen_position.Z / 3,
            health_factor = health_factor,
            health_color = self.health_color_1:lerp(self.health_color_2, health_factor),
            visible = utility.camera.isvisible(cframe.p, self._model, {camera, self._model})
        }
    end,
    _step = function(self, check, check_data)

        self:set_visible(false)
        self._components._chams.highlight.Enabled = false

        if not check then
            if self.arrow and check_data ~= nil and not check_data.screen_visible then
                local fill = self._components._drawings.arrow_fill
                local outline = self._components._drawings.arrow_outline
                local proj = camera.CFrame:PointToObjectSpace(check_data.cframe.p)
                local ang = math_atan2(proj.Z, proj.X)
                local dir = vector2_new(math_cos(ang), math_sin(ang))
                local a = (dir * self.arrow_radius * 0.5) + camera.ViewportSize / 2
                local b = a - utility.vector2.rotate(dir, math_rad(35)) * self.arrow_size
                local c = a - dir * (self.arrow_size / 1.525)
                local d = a - utility.vector2.rotate(dir, -math_rad(35)) * self.arrow_size
                fill.Visible = true
                fill.PointA = a
                fill.PointB = b
                fill.PointC = c
                fill.PointD = d
                outline.Visible = true
                outline.PointA = a
                outline.PointB = b
                outline.PointC = c
                outline.PointD = d

                if self.arrow_pulse and self.arrow_pulse_distance > check_data.distance then
                    local t = math_sin(math_rad((tick() * 175) % 180))
                    fill.Color = self.arrow_pulse_color
                    outline.Color = self.arrow_pulse_color
                    fill.Transparency = math_clamp(t, 0, 0.5)
                    outline.Transparency = t
                else
                    fill.Color  = self.arrow_color
                    outline.Color  = self.arrow_color
                    fill.Transparency = 0.5
                    outline.Transparency = 1
                end
            else
                self._components._drawings.arrow_fill.Visible = false
                self._components._drawings.arrow_outline.Visible = false
            end
            return
        end

        self._components._drawings.arrow_fill.Visible = false
        self._components._drawings.arrow_outline.Visible = false

        local factor   = 1 / (check_data.distance * math_tan(math_rad(camera.FieldOfView / 2)) * 2) * 1000
        local size     = vector2_new(math_clamp(math_floor(factor * 1.1), 3, 9999), math_clamp(math_floor(factor * 2), 6, 9999))--esp.get_box_size(check_data.screen_position, check_data.cframe * self._offset, self._size)
        local position = utility.vector2.floor(check_data.screen_position - (size / 2))
        local zindex   = (15000 - check_data.distance) - 16000

        local highlight     = self._components._chams.highlight
        local box           = self._components._drawings.box
        local box_outline   = self._components._drawings.box_outline
        local health_fill   = self._components._drawings.health_fill
        local health_border = self._components._drawings.health_border

        local text_data = self:get_text_data(check_data)
        local text_positions = {top = 0, bottom = 0, left = 0, right = 0}

        local force_color = (
            (self.targetcheck and esp.target == self) and self.target_color or
            (self.friendcheck and check_data.friend == true) and self.friend_color or
            (self.targetable_check and check_data.targetable == true) and self.targetable_color or
            (self.vischeck and check_data.visible) and self.vischeck_color or
            (self.notvischeck and not check_data.visible) and self.notvischeck_color
        )
        
        -- chams
        if highlight.Parent == nil then
            highlight:Destroy()
            highlight = esp.instance('Highlight')
            self._components._chams.highlight = highlight
        end

        highlight.Enabled = self.chams and check_data.distance < 500 and (self._model ~= nil or check_data.model ~= nil)

        if highlight.Enabled then
            highlight.Adornee = self._model or check_data.model
            highlight.Parent  = self._model or check_data.model
            highlight.FillColor = force_color or self.chams_color_1
            highlight.FillTransparency = self.chams_opacity_1
            highlight.OutlineColor = force_color or self.chams_color_2
            highlight.OutlineTransparency = self.chams_opacity_2 
            highlight.DepthMode = self.chams_occluded and 'Occluded' or 'AlwaysOnTop'
        end
        
        -- box
        box.Visible = self.box
        box_outline.Visible = self.box

        if self.box then
            box.Size = size
            box.Position = position
            box.Color = force_color or self.box_color
            box.ZIndex = zindex
    
            box_outline.Size = size
            box_outline.Position = position 
            box_outline.ZIndex = zindex - 1
        end

        -- health bar
        health_border.Visible = self.healthbar
        health_fill.Visible = self.healthbar

        if self.healthbar then
            local y = 1 + (size.Y - health_fill.Size.Y)
            text_positions.left = y
            health_border.Size = vector2_new(3, size.Y + 2)
            health_border.Position = position - vector2_new(5,1)
            health_border.ZIndex = zindex - 1
            health_fill.Size = vector2_new(1,check_data.health_factor * size.Y)
            health_fill.Position = health_border.Position + vector2_new(1,y)
            health_fill.Color = check_data.health_color 
            health_fill.ZIndex = zindex
        end

        -- text

        for flag, text_position in next, self.text_layout do
            local layout  = text_data[flag] or {}
            local drawing = self._components._text_drawings[flag]
            local visible = self.text and layout.enabled

            if not drawing then
                drawing = library:create('Text', {Size = 13, Font = 2, Outline = true})
                self._components._text_drawings[flag] = drawing
            end

            drawing.Visible = visible

            if not visible then 
                continue
            end

            drawing.Text = layout.text == nil and flag or tostring(layout.text)
            drawing.Color = layout.color or force_color or self.text_color
            drawing.Center = text_position == 'top' or text_position == 'bottom'
            drawing.ZIndex = zindex

            drawing.Position = position + (
                text_position == 'top'    and vector2_new(size.X / 2, - text_positions.top - 4 - drawing.TextBounds.Y) or
                text_position == 'bottom' and vector2_new(size.X / 2, size.Y + text_positions.bottom + 1) or
                text_position == 'left'   and vector2_new(-3 - drawing.TextBounds.X - (self.healthbar and 5 or 0), text_positions.left - 4) or
                text_position == 'right'  and vector2_new(size.X + 4, text_positions.right - 4)
            )

            text_positions[text_position] = text_positions[text_position] + 11

        end

        self._visible = true

    end
}, function(object)
    local object = setmetatable(object, esp_object)

    object._components = {
        _drawings = {
            box           = library:create('Square', {ZIndex = -15,  Thickness = 1}),
            box_outline   = library:create('Square', {ZIndex = -16,  Thickness = 3}),
            health_fill   = library:create('Square', {ZIndex = -15,  Filled = true}),
            health_border = library:create('Square', {ZIndex = -16,  Filled = true}),
            arrow_fill    = library:create('Quad',   {ZIndex = -14,  Filled = true, Transparency = 0.5}),
            arrow_outline = library:create('Quad',   {ZIndex = -13,  Thickness = 1})
        },
        _text_drawings = {},
        _chams = {
            highlight = esp.instance('Highlight'),
            adornees = {}
        }
    }

    object._stepped = library.signal.new()

    table.insert(object._connections, object._stepped:Connect(function(check, check_data)
        object:_step(check, check_data)
    end))

    table.insert(esp.objects, object)
    return object
end)

esp.define('text', {
    _cframe = CFrame.new(0,0,0),
    _visible = false,
    _connections = {},
    _properties = {
        enabled = false,
        showdistance = false,
        color = color3_new(1,1,1),
        text = '',
    },
    _check = function(self)
        if not self.enabled then
            return false
        end

        local distance = (camera.CFrame.p - self._cframe.p).magnitude
        if distance > self._group.maxdistance then
            return false
        end

        return true, distance
    end,
    _step = function(self, check, distance)

        if not check then
            self._drawing.Visible = check
            return
        end

        local screen_position, screen_visible = utility.camera.cframetoviewport(self._cframe)
    
        self._drawing.Visible = screen_visible
        self._drawing.Position = vector2_new(screen_position.X, screen_position.Y)
        self._drawing.Color = self.color
        self._drawing.Text = self.text .. (self.showdistance and '[' .. math_floor(distance) .. ']' or '')

    end
}, function(object)
    object._drawing = library:create('Text', {ZIndex = -20, Size = 13, Font = 2, Outline = true, Center = true})
    table.insert(esp.objects, object)
    return object
end)

function esp.new(flag, properties)
    local group = {
        _group = esp.create('group', properties)
    }

    function group:add(class, properties)
        local object = esp.create(class, properties)
        rawset(object, '_group', self._group)
        table.insert(self._group._objects, object)
        for i,v in next, self._group._properties do
            object[i] = v
        end
        return object
    end

    return setmetatable({}, {
        __index = function(self, idx)
            if group[idx] ~= nil then
                return group[idx]
            end
            return group._group[idx]
        end,
        __newindex = function(self, idx, val)
            group._group[idx] = val
        end
    })
end

library:connection(game:GetService('RunService').PostSimulation, function()
    if esp.autostep and tick() - esp.lastrender > 1 / esp.fps then
        esp.lastrender = tick()

        for _, object in next, esp.objects do
            if object._stepped ~= nil then
                local enabled = object._group._enabled and object.enabled

                if not enabled then
                    object._stepped:Fire(false)
                    continue
                end

                local check, check_data = object:_check()
                object._stepped:Fire(check, check_data)
            end
        end
    end
end)

getgenv().esp = esp
return esp

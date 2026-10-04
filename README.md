--// CloudUI - Custom Roblox UI Library
--// Rayfield-inspired layout, original implementation
--// Includes improved draggable bottom-right resize handle
--// Includes larger notifications
--// Includes larger paragraph/label text
--// No Lucide support
--// No emojis inside code

local CloudUI = {}

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")

local Player = Players.LocalPlayer

local Theme = {
    Background = Color3.fromRGB(18, 18, 22),
    Secondary = Color3.fromRGB(24, 24, 29),
    Element = Color3.fromRGB(31, 31, 38),
    Hover = Color3.fromRGB(40, 40, 48),
    Accent = Color3.fromRGB(120, 90, 255),
    Text = Color3.fromRGB(245, 245, 250),
    SubText = Color3.fromRGB(160, 160, 170),
    Border = Color3.fromRGB(50, 50, 60)
}

local function Create(class, properties)
    local object = Instance.new(class)

    for property, value in pairs(properties or {}) do
        object[property] = value
    end

    return object
end

local function Corner(parent, radius)
    return Create("UICorner", {
        Parent = parent,
        CornerRadius = UDim.new(0, radius)
    })
end

local function Stroke(parent, color, thickness)
    return Create("UIStroke", {
        Parent = parent,
        Color = color,
        Thickness = thickness or 1
    })
end

local function Tween(object, properties, duration)
    TweenService:Create(
        object,
        TweenInfo.new(
            duration or 0.2,
            Enum.EasingStyle.Quart,
            Enum.EasingDirection.Out
        ),
        properties
    ):Play()
end

function CloudUI:CreateWindow(settings)
    settings = settings or {}

    local Title = settings.Name or "Cloud UI"
    local Size = settings.Size or UDim2.fromOffset(620, 430)

    local MinSize = settings.MinSize or Vector2.new(400, 300)
    local MaxSize = settings.MaxSize or Vector2.new(1000, 750)

    local Gui = Create("ScreenGui", {
        Name = "CloudUI",
        ResetOnSpawn = false,
        ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    })

    Gui.Parent = Player:WaitForChild("PlayerGui")

    local Main = Create("Frame", {
        Parent = Gui,
        Size = Size,
        Position = UDim2.new(
            0.5,
            -Size.X.Offset / 2,
            0.5,
            -Size.Y.Offset / 2
        ),
        BackgroundColor3 = Theme.Background,
        BorderSizePixel = 0,
        ClipsDescendants = false,
        Active = true
    })

    Corner(Main, 12)
    Stroke(Main, Theme.Border, 1)

    ------------------------------------------------------------
    -- TOP BAR
    ------------------------------------------------------------

    local Top = Create("Frame", {
        Parent = Main,
        Size = UDim2.new(1, 0, 0, 55),
        BackgroundTransparency = 1,
        Active = true,
        ZIndex = 5
    })

    Create("TextLabel", {
        Parent = Top,
        Position = UDim2.fromOffset(18, 8),
        Size = UDim2.new(1, -80, 0, 25),
        BackgroundTransparency = 1,
        Text = Title,
        TextColor3 = Theme.Text,
        TextSize = 20,
        Font = Enum.Font.GothamBold,
        TextXAlignment = Enum.TextXAlignment.Left,
        ZIndex = 6
    })

    Create("TextLabel", {
        Parent = Top,
        Position = UDim2.fromOffset(18, 31),
        Size = UDim2.new(1, -80, 0, 18),
        BackgroundTransparency = 1,
        Text = settings.Subtitle or "Cloud UI",
        TextColor3 = Theme.SubText,
        TextSize = 11,
        Font = Enum.Font.Gotham,
        TextXAlignment = Enum.TextXAlignment.Left,
        ZIndex = 6
    })

    local Minimize = Create("TextButton", {
        Parent = Top,
        Position = UDim2.new(1, -45, 0, 12),
        Size = UDim2.fromOffset(30, 30),
        BackgroundColor3 = Theme.Element,
        Text = "—",
        TextColor3 = Theme.Text,
        TextSize = 18,
        Font = Enum.Font.GothamBold,
        AutoButtonColor = false,
        ZIndex = 10
    })

    Corner(Minimize, 7)

    ------------------------------------------------------------
    -- SIDEBAR
    ------------------------------------------------------------

    local Sidebar = Create("Frame", {
        Parent = Main,
        Position = UDim2.fromOffset(10, 65),
        Size = UDim2.new(0, 145, 1, -75),
        BackgroundColor3 = Theme.Secondary,
        BorderSizePixel = 0,
        ZIndex = 2
    })

    Corner(Sidebar, 9)

    local TabList = Create("ScrollingFrame", {
        Parent = Sidebar,
        Position = UDim2.fromOffset(6, 8),
        Size = UDim2.new(1, -12, 1, -16),
        BackgroundTransparency = 1,
        BorderSizePixel = 0,
        ScrollBarThickness = 2,
        ScrollBarImageColor3 = Theme.Accent,
        CanvasSize = UDim2.new(),
        ZIndex = 3
    })

    Create("UIListLayout", {
        Parent = TabList,
        Padding = UDim.new(0, 6),
        SortOrder = Enum.SortOrder.LayoutOrder
    })

    ------------------------------------------------------------
    -- CONTENT
    ------------------------------------------------------------

    local Content = Create("Frame", {
        Parent = Main,
        Position = UDim2.fromOffset(165, 65),
        Size = UDim2.new(1, -175, 1, -75),
        BackgroundTransparency = 1,
        ZIndex = 2
    })

    local Pages = {}
    local Tabs = {}
    local FirstTab = nil

    ------------------------------------------------------------
    -- TAB SYSTEM
    ------------------------------------------------------------

    function CloudUI:AddTab(tabSettings)
        tabSettings = tabSettings or {}

        local TabName = tabSettings.Name or "Tab"

        local Button = Create("TextButton", {
            Parent = TabList,
            Size = UDim2.new(1, 0, 0, 38),
            BackgroundColor3 = Theme.Element,
            BackgroundTransparency = 1,
            Text = TabName,
            TextColor3 = Theme.SubText,
            TextSize = 13,
            Font = Enum.Font.GothamMedium,
            AutoButtonColor = false
        })

        Corner(Button, 7)

        local Page = Create("ScrollingFrame", {
            Parent = Content,
            Size = UDim2.fromScale(1, 1),
            BackgroundTransparency = 1,
            BorderSizePixel = 0,
            ScrollBarThickness = 3,
            ScrollBarImageColor3 = Theme.Accent,
            Visible = false,
            CanvasSize = UDim2.new()
        })

        local Layout = Create("UIListLayout", {
            Parent = Page,
            Padding = UDim.new(0, 8),
            SortOrder = Enum.SortOrder.LayoutOrder
        })

        Layout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
            Page.CanvasSize = UDim2.new(
                0,
                0,
                0,
                Layout.AbsoluteContentSize.Y + 10
            )
        end)

        local Tab = {}

        function Tab:Select()
            for _, page in pairs(Pages) do
                page.Visible = false
            end

            for _, button in pairs(Tabs) do
                Tween(button, {
                    BackgroundTransparency = 1,
                    TextColor3 = Theme.SubText
                })
            end

            Page.Visible = true

            Tween(Button, {
                BackgroundTransparency = 0,
                BackgroundColor3 = Theme.Accent,
                TextColor3 = Theme.Text
            })
        end

        --------------------------------------------------------
        -- SECTION
        --------------------------------------------------------

        function Tab:AddSection(text)
            return Create("TextLabel", {
                Parent = Page,
                Size = UDim2.new(1, -8, 0, 25),
                BackgroundTransparency = 1,
                Text = text,
                TextColor3 = Theme.Text,
                TextSize = 14,
                Font = Enum.Font.GothamBold,
                TextXAlignment = Enum.TextXAlignment.Left
            })
        end

        --------------------------------------------------------
        -- BUTTON
        --------------------------------------------------------

        function Tab:AddButton(settings)
            settings = settings or {}

            local ButtonFrame = Create("TextButton", {
                Parent = Page,
                Size = UDim2.new(1, -8, 0, 45),
                BackgroundColor3 = Theme.Element,
                Text = settings.Name or "Button",
                TextColor3 = Theme.Text,
                TextSize = 13,
                Font = Enum.Font.GothamMedium,
                AutoButtonColor = false
            })

            Corner(ButtonFrame, 8)
            Stroke(ButtonFrame, Theme.Border)

            ButtonFrame.MouseEnter:Connect(function()
                Tween(ButtonFrame, {
                    BackgroundColor3 = Theme.Hover
                })
            end)

            ButtonFrame.MouseLeave:Connect(function()
                Tween(ButtonFrame, {
                    BackgroundColor3 = Theme.Element
                })
            end)

            ButtonFrame.Activated:Connect(function()
                if settings.Callback then
                    settings.Callback()
                end
            end)

            return ButtonFrame
        end

        --------------------------------------------------------
        -- TOGGLE
        --------------------------------------------------------

        function Tab:AddToggle(settings)
            settings = settings or {}

            local Enabled = settings.CurrentValue or false

            local Button = Create("TextButton", {
                Parent = Page,
                Size = UDim2.new(1, -8, 0, 45),
                BackgroundColor3 = Theme.Element,
                Text = settings.Name or "Toggle",
                TextColor3 = Theme.Text,
                TextSize = 13,
                Font = Enum.Font.GothamMedium,
                TextXAlignment = Enum.TextXAlignment.Left,
                AutoButtonColor = false
            })

            Corner(Button, 8)
            Stroke(Button, Theme.Border)

            local Indicator = Create("Frame", {
                Parent = Button,
                Position = UDim2.new(1, -48, 0.5, -10),
                Size = UDim2.fromOffset(38, 20),
                BackgroundColor3 = Theme.Background
            })

            Corner(Indicator, 10)

            local Circle = Create("Frame", {
                Parent = Indicator,
                Position = UDim2.fromOffset(3, 3),
                Size = UDim2.fromOffset(14, 14),
                BackgroundColor3 = Theme.SubText
            })

            Corner(Circle, 20)

            local function Update()
                if Enabled then
                    Tween(Indicator, {
                        BackgroundColor3 = Theme.Accent
                    })

                    Tween(Circle, {
                        Position = UDim2.new(1, -17, 0, 3),
                        BackgroundColor3 = Theme.Text
                    })
                else
                    Tween(Indicator, {
                        BackgroundColor3 = Theme.Background
                    })

                    Tween(Circle, {
                        Position = UDim2.fromOffset(3, 3),
                        BackgroundColor3 = Theme.SubText
                    })
                end
            end

            Button.Activated:Connect(function()
                Enabled = not Enabled

                Update()

                if settings.Callback then
                    settings.Callback(Enabled)
                end
            end)

            Update()

            return {
                Set = function(_, value)
                    Enabled = value
                    Update()

                    if settings.Callback then
                        settings.Callback(Enabled)
                    end
                end
            }
        end

        --------------------------------------------------------
        -- LABEL / PARAGRAPH
        --------------------------------------------------------

        function Tab:AddLabel(text)
            return Create("TextLabel", {
                Parent = Page,
                Size = UDim2.new(1, -8, 0, 34),
                BackgroundTransparency = 1,
                Text = text,
                TextColor3 = Theme.SubText,
                TextSize = 14,
                Font = Enum.Font.Gotham,
                TextXAlignment = Enum.TextXAlignment.Left,
                TextYAlignment = Enum.TextYAlignment.Center
            })
        end

        --------------------------------------------------------
        -- TEXTBOX
        --------------------------------------------------------

        function Tab:AddTextbox(settings)
            settings = settings or {}

            local Box = Create("TextBox", {
                Parent = Page,
                Size = UDim2.new(1, -8, 0, 42),
                BackgroundColor3 = Theme.Element,
                PlaceholderText = settings.PlaceholderText or "Enter text...",
                Text = settings.Text or "",
                TextColor3 = Theme.Text,
                PlaceholderColor3 = Theme.SubText,
                TextSize = 13,
                Font = Enum.Font.Gotham,
                ClearTextOnFocus = false
            })

            Corner(Box, 8)
            Stroke(Box, Theme.Border)

            Box.FocusLost:Connect(function()
                if settings.Callback then
                    settings.Callback(Box.Text)
                end
            end)

            return Box
        end

        Button.Activated:Connect(function()
            Tab:Select()
        end)

        table.insert(Pages, Page)
        table.insert(Tabs, Button)

        if not FirstTab then
            FirstTab = Tab
            Tab:Select()
        end

        return Tab
    end

    ------------------------------------------------------------
    -- WINDOW DRAGGING
    ------------------------------------------------------------

    local dragging = false
    local dragStart
    local startPosition

    Top.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
            or input.UserInputType == Enum.UserInputType.Touch then

            dragging = true
            dragStart = input.Position
            startPosition = Main.Position

            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if not dragging then
            return
        end

        if input.UserInputType ~= Enum.UserInputType.MouseMovement
            and input.UserInputType ~= Enum.UserInputType.Touch then
            return
        end

        local delta = input.Position - dragStart

        Main.Position = UDim2.new(
            startPosition.X.Scale,
            startPosition.X.Offset + delta.X,
            startPosition.Y.Scale,
            startPosition.Y.Offset + delta.Y
        )
    end)

    ------------------------------------------------------------
    -- IMPROVED RESIZE HANDLE
    ------------------------------------------------------------

    local ResizeHandle = Create("TextButton", {
        Parent = Main,
        AnchorPoint = Vector2.new(1, 1),
        Position = UDim2.new(1, -2, 1, -2),
        Size = UDim2.fromOffset(32, 32),
        BackgroundTransparency = 1,
        Text = "",
        AutoButtonColor = false,
        Active = true,
        ZIndex = 50
    })

    local ResizeLine1 = Create("Frame", {
        Parent = ResizeHandle,
        AnchorPoint = Vector2.new(0.5, 0.5),
        Position = UDim2.new(0.5, 2, 0.5, 3),
        Size = UDim2.fromOffset(3, 12),
        Rotation = 45,
        BackgroundColor3 = Theme.SubText,
        BorderSizePixel = 0,
        ZIndex = 51
    })

    local ResizeLine2 = Create("Frame", {
        Parent = ResizeHandle,
        AnchorPoint = Vector2.new(0.5, 0.5),
        Position = UDim2.new(0.5, 5, 0.5, 0),
        Size = UDim2.fromOffset(3, 18),
        Rotation = 45,
        BackgroundColor3 = Theme.SubText,
        BorderSizePixel = 0,
        ZIndex = 51
    })

    local resizing = false
    local resizeInput = nil
    local resizeStart = nil
    local resizeStartSize = nil

    local function SetResizeVisual(active)
        local color = active and Theme.Text or Theme.SubText

        Tween(ResizeLine1, {
            BackgroundColor3 = color
        }, 0.15)

        Tween(ResizeLine2, {
            BackgroundColor3 = color
        }, 0.15)
    end

    ResizeHandle.MouseEnter:Connect(function()
        SetResizeVisual(true)
    end)

    ResizeHandle.MouseLeave:Connect(function()
        if not resizing then
            SetResizeVisual(false)
        end
    end)

    ResizeHandle.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
            or input.UserInputType == Enum.UserInputType.Touch then

            resizing = true
            resizeInput = input
            resizeStart = input.Position
            resizeStartSize = Main.AbsoluteSize

            SetResizeVisual(true)
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if not resizing then
            return
        end

        if input == resizeInput
            or input.UserInputType == Enum.UserInputType.MouseMovement
            or input.UserInputType == Enum.UserInputType.Touch then

            local delta = input.Position - resizeStart

            local newWidth = math.clamp(
                resizeStartSize.X + delta.X,
                MinSize.X,
                MaxSize.X
            )

            local newHeight = math.clamp(
                resizeStartSize.Y + delta.Y,
                MinSize.Y,
                MaxSize.Y
            )

            Main.Size = UDim2.fromOffset(
                newWidth,
                newHeight
            )
        end
    end)

    UserInputService.InputEnded:Connect(function(input)
        if not resizing then
            return
        end

        if input == resizeInput
            or input.UserInputType == Enum.UserInputType.MouseButton1
            or input.UserInputType == Enum.UserInputType.Touch then

            resizing = false
            resizeInput = nil
            SetResizeVisual(false)
        end
    end)

    ------------------------------------------------------------
    -- MINIMIZE
    ------------------------------------------------------------

    local minimized = false
    local PreviousSize = Size

    Minimize.Activated:Connect(function()
        minimized = not minimized

        if minimized then
            PreviousSize = Main.Size

            Tween(Main, {
                Size = UDim2.fromOffset(
                    PreviousSize.X.Offset,
                    55
                )
            })
        else
            Tween(Main, {
                Size = PreviousSize
            })
        end
    end)

    ------------------------------------------------------------
    -- NOTIFICATIONS
    ------------------------------------------------------------

    function CloudUI:Notify(settings)
        settings = settings or {}

        local NotificationTitle = settings.Title or "Cloud UI"
        local NotificationContent = settings.Content or ""
        local Duration = tonumber(settings.Duration) or 4

        local NotificationWidth = 360
        local NotificationHeight = 100

        local Notification = Create("Frame", {
            Parent = Gui,
            Size = UDim2.fromOffset(
                NotificationWidth,
                NotificationHeight
            ),
            Position = UDim2.new(1, 25, 1, -120),
            BackgroundColor3 = Theme.Secondary,
            BorderSizePixel = 0,
            ClipsDescendants = true,
            ZIndex = 100
        })

        Corner(Notification, 12)
        Stroke(Notification, Theme.Border, 1)

        local Accent = Create("Frame", {
            Parent = Notification,
            Size = UDim2.fromOffset(5, NotificationHeight),
            Position = UDim2.fromOffset(0, 0),
            BackgroundColor3 = Theme.Accent,
            BorderSizePixel = 0,
            ZIndex = 101
        })

        Corner(Accent, 5)

        local TitleLabel = Create("TextLabel", {
            Parent = Notification,
            Position = UDim2.fromOffset(20, 12),
            Size = UDim2.new(1, -65, 0, 25),
            BackgroundTransparency = 1,
            Text = NotificationTitle,
            TextColor3 = Theme.Text,
            TextSize = 16,
            Font = Enum.Font.GothamBold,
            TextXAlignment = Enum.TextXAlignment.Left,
            TextYAlignment = Enum.TextYAlignment.Center,
            ZIndex = 101
        })

        local ContentLabel = Create("TextLabel", {
            Parent = Notification,
            Position = UDim2.fromOffset(20, 40),
            Size = UDim2.new(1, -40, 0, 38),
            BackgroundTransparency = 1,
            Text = NotificationContent,
            TextColor3 = Theme.SubText,
            TextSize = 13,
            Font = Enum.Font.Gotham,
            TextWrapped = true,
            TextXAlignment = Enum.TextXAlignment.Left,
            TextYAlignment = Enum.TextYAlignment.Top,
            ZIndex = 101
        })

        local Close = Create("TextButton", {
            Parent = Notification,
            Position = UDim2.new(1, -38, 0, 9),
            Size = UDim2.fromOffset(27, 27),
            BackgroundTransparency = 1,
            Text = "×",
            TextColor3 = Theme.SubText,
            TextSize = 20,
            Font = Enum.Font.GothamBold,
            AutoButtonColor = false,
            ZIndex = 102
        })

        Close.MouseEnter:Connect(function()
            Tween(Close, {
                TextColor3 = Theme.Text
            }, 0.15)
        end)

        Close.MouseLeave:Connect(function()
            Tween(Close, {
                TextColor3 = Theme.SubText
            }, 0.15)
        end)

        local ProgressBackground = Create("Frame", {
            Parent = Notification,
            Position = UDim2.new(0, 0, 1, -4),
            Size = UDim2.new(1, 0, 0, 4),
            BackgroundColor3 = Theme.Element,
            BorderSizePixel = 0,
            ZIndex = 101
        })

        local Progress = Create("Frame", {
            Parent = ProgressBackground,
            Size = UDim2.fromScale(1, 1),
            BackgroundColor3 = Theme.Accent,
            BorderSizePixel = 0,
            ZIndex = 102
        })

        local Closed = false

        local function CloseNotification()
            if Closed then
                return
            end

            Closed = true

            Tween(Notification, {
                Position = UDim2.new(
                    1,
                    25,
                    1,
                    -120
                )
            }, 0.25)

            task.delay(0.3, function()
                if Notification and Notification.Parent then
                    Notification:Destroy()
                end
            end)
        end

        Close.Activated:Connect(CloseNotification)

        Tween(Notification, {
            Position = UDim2.new(
                1,
                -(NotificationWidth + 15),
                1,
                -120
            )
        }, 0.35)

        Tween(Progress, {
            Size = UDim2.new(0, 0, 1, 0)
        }, Duration)

        task.delay(Duration, function()
            CloseNotification()
        end)

        return {
            Close = CloseNotification,

            SetTitle = function(_, NewTitle)
                TitleLabel.Text = tostring(NewTitle)
            end,

            SetContent = function(_, NewContent)
                ContentLabel.Text = tostring(NewContent)
            end,

            SetDuration = function(_, NewDuration)
                NewDuration = tonumber(NewDuration) or Duration

                Tween(Progress, {
                    Size = UDim2.fromScale(1, 1)
                }, 0)

                Tween(Progress, {
                    Size = UDim2.new(0, 0, 1, 0)
                }, NewDuration)

                task.delay(NewDuration, function()
                    CloseNotification()
                end)
            end
        }
    end

    ------------------------------------------------------------
    -- RETURN WINDOW
    ------------------------------------------------------------

    return {
        AddTab = function(_, settings)
            return CloudUI:AddTab(settings)
        end,

        Notify = function(_, settings)
            return CloudUI:Notify(settings)
        end,

        Destroy = function()
            Gui:Destroy()
        end,

        SetSize = function(_, width, height)
            local newWidth = math.clamp(
                tonumber(width) or Size.X.Offset,
                MinSize.X,
                MaxSize.X
            )

            local newHeight = math.clamp(
                tonumber(height) or Size.Y.Offset,
                MinSize.Y,
                MaxSize.Y
            )

            Main.Size = UDim2.fromOffset(
                newWidth,
                newHeight
            )
        end,

        GetSize = function()
            return Main.AbsoluteSize
        end
    }
end

return CloudUI

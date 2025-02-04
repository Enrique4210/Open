1. VERIFICACION DE TANKES CONECTADOS A LA COMPUTADORA

if not component.list("tank_controller") then
    print("✗ Error: No hay tank_controller conectados")
    return
end

for address, _ in pairs(component.list("tank_controller")) do
    print("Tanque detectado en: " .. address)
end

2. DETECCION DE TANKES SEGUN SU ID INDEPENDIENTE Y DIRECCION DISPUESTO

local component = require("component")
local sides = require("sides")

-- Direcciones de los tanques para cada lado
local tankAddresses = {
    [sides.north] = "ID DEL TANK_CONTROLLER",  -- Lado Norte
    [sides.west]  = "ID DEL TANK_CONTROLLER",   -- Lado Oeste
    [sides.south] = "ID DEL TANK_CONTROLLER",   -- Lado Sur
    [sides.east]  = "ID DEL TANK_CONTROLLER"    -- Lado Este
}

-- Función para obtener y mostrar la información del tanque
local function showTankInfo(tank, side, name)
    local fluid = tank.getFluidInTank(side)[1]
    if fluid then
        print("✦ Tanque " .. name)
        print("  Líquido: " .. (fluid.name or "Desconocido"))
        print("  Cantidad: " .. fluid.amount .. "/" .. tank.getTankCapacity(side))
    else
        print("✦ Tanque " .. name .. ": Vacío o no conectado correctamente.")
    end
end

-- Iterar sobre los lados definidos y obtener la información de cada tanque
for side, address in pairs(tankAddresses) do
    local tank = component.proxy(address)  -- Obtener el tanque usando la dirección
    if tank then
        showTankInfo(tank, side, sides[side])  -- Mostrar la información
    else
        print("✗ Error: No se pudo acceder al tanque en el lado " .. sides[side])
    end
end

3. OBTENCION DE VALORES DE LOS TANKES EN TIEMPO REAL

local component = require("component")
local sides = require("sides")
local os = require("os")
local term = require("term")

-- Direcciones de los tanques para cada lado
local tankAddresses = {
    [sides.north] = "ID DEL TANK_CONTROLLER",  -- Lado Norte
    [sides.west]  = "ID DEL TANK_CONTROLLER",   -- Lado Oeste
    [sides.south] = "ID DEL TANK_CONTROLLER",   -- Lado Sur
    [sides.east]  = "ID DEL TANK_CONTROLLER"    -- Lado Este
}

-- Función para crear una barra de progreso vertical gruesa
local function createVerticalProgressBar(current, max, height)
    local ratio = current / max
    local filledHeight = math.floor(ratio * height)
    local bar = ""
    for i = height, 1, -1 do
        if i <= filledHeight then
            bar = bar .. "███\n"  -- Bloques gruesos para la parte llena
        else
            bar = bar .. "   \n"  -- Espacios en blanco para la parte vacía
        end
    end
    return bar
end

-- Función para mostrar la información de los tanques
local function displayTankInfo()
    -- Variables para acumular la cantidad total y la capacidad total
    local totalAmount = 0
    local totalCapacity = 0

    -- Posición inicial del cursor para la información de los tanques
    local line = 2  -- Comenzar en la línea 2 (la línea 1 es para el título)

    -- Iterar sobre los lados definidos y obtener la información de cada tanque
    for side, address in pairs(tankAddresses) do
        term.setCursor(1, line)  -- Mover el cursor a la línea correspondiente
        local tank = component.proxy(address)  -- Obtener el tanque usando la dirección
        if tank then
            local fluid = tank.getFluidInTank(side)[1]
            if fluid then
                print("✦ Tanque " .. sides[side] .. ": " .. (fluid.name or "Desconocido") .. " " .. fluid.amount .. "/" .. tank.getTankCapacity(side))
                totalAmount = totalAmount + fluid.amount
                totalCapacity = totalCapacity + tank.getTankCapacity(side)
            else
                print("✦ Tanque " .. sides[side] .. ": Vacío o no conectado correctamente.")
            end
        else
            print("✗ Error: No se pudo acceder al tanque en el lado " .. sides[side])
        end
        line = line + 1  -- Mover a la siguiente línea
    end

    -- Mostrar el total acumulado
    term.setCursor(1, line + 1)  -- Mover el cursor a la línea del total
    print("\n✪✪✪ TOTAL EN TODOS LOS TANQUES ✪✪✪")
    print(string.rep("=", 50))
    print(string.format("✦ Cantidad total: %d/%d", totalAmount, totalCapacity))

    -- Calcular el porcentaje
    local percentage = (totalAmount / totalCapacity) * 100
    print(string.format("✦ Porcentaje: %.2f%%", percentage))

    -- Crear y mostrar la barra de progreso vertical gruesa
    local progressBar = createVerticalProgressBar(totalAmount, totalCapacity, 10)  -- Altura de 10 líneas
    print("\nBarra de progreso:")
    print(progressBar)
    print(string.rep("=", 50))
end

-- Función para inicializar la pantalla
local function initializeScreen()
    term.clear()  -- Limpiar la pantalla al inicio
    term.setCursor(1, 1)
    print("✪✪✪ MONITOREO DE TANQUES EN TIEMPO REAL ✪✪✪")
end

-- Inicializar la pantalla
initializeScreen()

-- Bucle principal para monitorear en tiempo real
while true do
    displayTankInfo()  -- Mostrar la información de los tanques
    os.sleep(0.5)  -- Esperar 0.5 segundos antes de la siguiente actualización
end




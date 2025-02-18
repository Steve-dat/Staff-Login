#include <a_samp>
#include <dini> // Para manejar archivos .ini

#define MAX_PLAYERS 50
#define DATA_PATH "Usuarios/%s.ini" // Ruta donde se guardarán los datos de los usuarios

// Niveles de staff
#define LEVEL_PLAYER 0
#define LEVEL_MODERATOR 1
#define LEVEL_ADMIN 2

new PlayerInfo[MAX_PLAYERS][pLevel]; // Almacenar nivel de staff del jugador

public OnGameModeInit() {
    print("Sistema de staff iniciado.");
    return 1;
}

public OnPlayerConnect(playerid) {
    new playerName[MAX_PLAYER_NAME];
    GetPlayerName(playerid, playerName, sizeof(playerName));
    
    new filePath[128];
    format(filePath, sizeof(filePath), DATA_PATH, playerName);
    
    if (fexist(filePath)) {
        PlayerInfo[playerid][pLevel] = dini_Int(filePath, "Level");
        SendClientMessage(playerid, -1, "Bienvenido de vuelta, staff!");
    } else {
        PlayerInfo[playerid][pLevel] = LEVEL_PLAYER;
        SendClientMessage(playerid, -1, "Bienvenido! Regístrate como staff si es necesario.");
    }
    return 1;
}

// Comando para establecer nivel de staff (solo accesible por administradores)
CMD:setlevel(playerid, params[]) {
    if (PlayerInfo[playerid][pLevel] != LEVEL_ADMIN) {
        SendClientMessage(playerid, -1, "No tienes permiso para usar este comando.");
        return 1;
    }

    new targetID, level;
    if (sscanf(params, "ui", targetID, level)) {
        SendClientMessage(playerid, -1, "USO: /setlevel [ID jugador] [nivel]");
        return 1;
    }

    if (!IsPlayerConnected(targetID)) {
        SendClientMessage(playerid, -1, "El jugador no está conectado.");
        return 1;
    }

    if (level < LEVEL_PLAYER || level > LEVEL_ADMIN) {
        SendClientMessage(playerid, -1, "Nivel inválido. Usa 0 (jugador), 1 (moderador) o 2 (admin).");
        return 1;
    }

    PlayerInfo[targetID][pLevel] = level;

    new playerName[MAX_PLAYER_NAME];
    GetPlayerName(targetID, playerName, sizeof(playerName));

    new filePath[128];
    format(filePath, sizeof(filePath), DATA_PATH, playerName);

    dini_Create(filePath);
    dini_IntSet(filePath, "Level", level);

    SendClientMessage(playerid, -1, "Nivel de staff actualizado.");
    SendClientMessage(targetID, -1, "Tu nivel de staff ha sido actualizado.");
    return 1;
}

// Comando para expulsar a un jugador (accesible por moderadores y administradores)
CMD:kick(playerid, params[]) {
    if (PlayerInfo[playerid][pLevel] < LEVEL_MODERATOR) {
        SendClientMessage(playerid, -1, "No tienes permiso para usar este comando.");
        return 1;
    }

    new targetID, reason[64];
    if (sscanf(params, "us[64]", targetID, reason)) {
        SendClientMessage(playerid, -1, "USO: /kick [ID jugador] [razón]");
        return 1;
    }

    if (!IsPlayerConnected(targetID)) {
        SendClientMessage(playerid, -1, "El jugador no está conectado.");
        return 1;
    }

    new targetName[MAX_PLAYER_NAME];
    GetPlayerName(targetID, targetName, sizeof(targetName));

    new senderName[MAX_PLAYER_NAME];
    GetPlayerName(playerid, senderName, sizeof(senderName));

    new message[128];
    format(message, sizeof(message), "%s ha sido expulsado por %s. Razón: %s", targetName, senderName, reason);
    SendClientMessageToAll(-1, message);

    Kick(targetID);
    return 1;
}

// Comando para silenciar a un jugador (accesible por moderadores y administradores)
CMD:mute(playerid, params[]) {
    if (PlayerInfo[playerid][pLevel] < LEVEL_MODERATOR) {
        SendClientMessage(playerid, -1, "No tienes permiso para usar este comando.");
        return 1;
    }

    new targetID;
    if (sscanf(params, "u", targetID)) {
        SendClientMessage(playerid, -1, "USO: /mute [ID jugador]");
        return 1;
    }

    if (!IsPlayerConnected(targetID)) {
        SendClientMessage(playerid, -1, "El jugador no está conectado.");
        return 1;
    }

    TogglePlayerChat(targetID, false);

    new targetName[MAX_PLAYER_NAME];
    GetPlayerName(targetID, targetName, sizeof(targetName));

    new message[128];
    format(message, sizeof(message), "%s ha sido silenciado.", targetName);
    SendClientMessageToAll(-1, message);

    return 1;
}

// Comando para desilenciar a un jugador (accesible por moderadores y administradores)
CMD:unmute(playerid, params[]) {
    if (PlayerInfo[playerid][pLevel] < LEVEL_MODERATOR) {
        SendClientMessage(playerid, -1, "No tienes permiso para usar este comando.");
        return 1;
    }

    new targetID;
    if (sscanf(params, "u", targetID)) {
        SendClientMessage(playerid, -1, "USO: /unmute [ID jugador]");
        return 1;
    }

    if (!IsPlayerConnected(targetID)) {
        SendClientMessage(playerid, -1, "El jugador no está conectado.");
        return 1;
    }

    TogglePlayerChat(targetID, true);

    new targetName[MAX_PLAYER_NAME];
    GetPlayerName(targetID, targetName, sizeof(targetName));

    new message[128];
    format(message, sizeof(message), "%s ha sido desilenciado.", targetName);
    SendClientMessageToAll(-1, message);

    return 1;
}

// Comando para teletransportarse (accesible por administradores)
CMD:tp(playerid, params[]) {
    if (PlayerInfo[playerid][pLevel] < LEVEL_ADMIN) {
        SendClientMessage(playerid, -1, "No tienes permiso para usar este comando.");
        return 1;
    }

    new Float:x, Float:y, Float:z;
    if (sscanf(params, "fff", x, y, z)) {
        SendClientMessage(playerid, -1, "USO: /tp [X] [Y] [Z]");
        return 1;
    }

    SetPlayerPos(playerid, x, y, z);
    SendClientMessage(playerid, -1, "Te has teletransportado.");
    return 1;
}

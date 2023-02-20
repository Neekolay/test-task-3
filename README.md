# test-task-3
Task: to refactor this code, optimize, make it more readable:

```java
void processTask(ChannelHandlerContext ctx) {
    InetSocketAddress lineAddress = new InetSocketAddress(getIpAddress(), getUdpPort());
    CommandType typeToRemove;
    for (Command currentCommand : getAllCommands()) {
        if (currentCommand.getCommandType() == CommandType.REBOOT_CHANNEL) {
            if (!currentCommand.isAttemptsNumberExhausted()) {
                if (currentCommand.isTimeToSend()) {
                    sendCommandToContext(ctx, lineAddress, currentCommand.getCommandText());

                    try {
                        AdminController.getInstance().processUssdMessage(
                                new DblIncomeUssdMessage(lineAddress.getHostName(), lineAddress.getPort(), 0, EnumGoip.getByModel(getGoipModel()), currentCommand.getCommandText()), false);
                    } catch (Exception ignored) {
                    }
                    currentCommand.setSendDate(new Date());
                    Log.ussd.write(String.format("sent: ip: %s; порт: %d; %s",
                            lineAddress.getHostString(), lineAddress.getPort(), currentCommand.getCommandText()));
                    currentCommand.incSendCounter();
                }
            } else {
                typeToRemove = currentCommand.getCommandType();
                deleteCommand(typeToRemove);
            }
        } else {
            if (!currentCommand.isAttemptsNumberExhausted()) {
                sendCommandToContext(ctx, lineAddress, currentCommand.getCommandText());

                try {
                    AdminController.getInstance().processUssdMessage(
                            new DblIncomeUssdMessage(lineAddress.getHostName(), lineAddress.getPort(), 0, EnumGoip.getByModel(getGoipModel()), currentCommand.getCommandText()), false);
                } catch (Exception ignored) {
                }
                Log.ussd.write(String.format("sent: ip: %s; порт: %d; %s",
                        lineAddress.getHostString(), lineAddress.getPort(), currentCommand.getCommandText()));
                currentCommand.setSendDate(new Date());
                currentCommand.incSendCounter();
            } else {
                typeToRemove = currentCommand.getCommandType();
                deleteCommand(typeToRemove);
            }
        }
    }
    sendKeepAliveOkAndFlush(ctx);
}
```

## Solution
```java
void processTask(ChannelHandlerContext ctx) {
    InetSocketAddress lineAddress = new InetSocketAddress(getIpAddress(), getUdpPort());
    for (Command currentCommand : getAllCommands()) {

        if (currentCommand.isAttemptsNumberExhausted() ||
                (currentCommand.getCommandType() == CommandType.REBOOT_CHANNEL && !currentCommand.isTimeToSend())) {
            deleteCommand(currentCommand.getCommandType());
            continue;
        }

        sendCommandToContext(ctx, lineAddress, currentCommand.getCommandText());
        try {
            var dblIncomeUssdMessage = new DblIncomeUssdMessage(lineAddress.getHostName(), lineAddress.getPort()
                                                                0, EnumGoip.getByModel(getGoipModel()),
                                                                currentCommand.getCommandText());
            AdminController.getInstance().processUssdMessage(dblIncomeUssdMessage, false);
        } catch (Exception ignored) {
        }

        currentCommand.setSendDate(new Date());
        currentCommand.incSendCounter();
        Log.ussd.write(String.format("sent: ip: %s; порт: %d; %s",
                lineAddress.getHostString(), lineAddress.getPort(), currentCommand.getCommandText()));

    }
    sendKeepAliveOkAndFlush(ctx);
}

```


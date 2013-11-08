Processing_proj
===============

project for processing
This is the project file for processing project
import java.util.ArrayList;
import java.util.HashMap;
import java.util.Map;
import java.util.Iterator;

import processing.core.*;
import SimpleOpenNI.*;

public class Handson extends PApplet {

	SimpleOpenNI context;

	int handVecListSize = 20;

	Map<Integer, ArrayList<PVector>> handPathList = new HashMap<Integer, ArrayList<PVector>>();

	int[] userClr = new int[] { color(255, 0, 0),

	color(0, 255, 0),

	color(0, 0, 255),

	color(255, 255, 0),

	color(255, 0, 255),

	color(0, 255, 255)

	};

	public void setup()

	{

		// frameRate(200);

		size(1024, 768);

		context = new SimpleOpenNI(this);

		if (context.isInit() == false)

		{

			println("Can't init SimpleOpenNI, maybe the camera is not connected!");

			exit();

			return;

		}

		// enable depthMap generation

		context.enableDepth();

		// disable mirror

		context.setMirror(true);

		// enable hands + gesture generation

		// context.enableGesture();

		context.enableHand();

		// add focus gestures / here i do have some problems on the mac, i only
		// recognize raiseHand ? Maybe cpu performance ?

		context.startGesture(SimpleOpenNI.GESTURE_WAVE);

		// set how smooth the hand capturing should be

		// context.setSmoothingHands(.5);
		smooth();
		noStroke();

	}

	public void draw()

	{

		// update the cam
		background(0);
		context.update();
		drawHUD();
		//translate(width / 2, height / 2);
	

		

		// draw the tracked hands

		if (handPathList.size() > 0)

		{

			Iterator itr = handPathList.entrySet().iterator();

			while (itr.hasNext())

			{

				Map.Entry mapEntry = (Map.Entry) itr.next();

				int handId = (Integer) mapEntry.getKey();

				ArrayList<PVector> vecList = (ArrayList<PVector>) mapEntry
						.getValue();

				PVector p;

				PVector p2d = new PVector();

				//stroke(userClr[(handId - 1) % userClr.length]);

				//noFill();

				//strokeWeight(1);

				Iterator<PVector> itrVec = vecList.iterator();

				beginShape();

				while (itrVec.hasNext())

				{

					p = (PVector) itrVec.next();

					context.convertRealWorldToProjective(p, p2d);

					//vertex(p2d.x, p2d.y);
					fill(200,0,0);
                     drawBall(p2d,handId);
				}

				endShape();

				//stroke(userClr[(handId - 1) % userClr.length]);

				strokeWeight(4);

				p = vecList.get(0);

				context.convertRealWorldToProjective(p, p2d);

				point(p2d.x, p2d.y);

			}

		}

	}
	int num = 60;
	int x[] = new int[num];
	int y[] = new int[num];
	
	public void drawBall(PVector v2, int handId) {
		//PVector v1 = new PVector();
		pushMatrix();
				//pushMatrix();

		
		for (int i = x.length-1; i > 0; i--) {
			x[i] = x[i-1];
			y[i] = y[i-1];
			//z[i] = z[i-1];
			}
		x[0] = (int) v2.x;
		y[0] = (int) v2.y;
		
		for (int i = 0; i < x.length; i++) {
			
			//i=userClr[(handId - 1) % userClr.length];
			fill(i * 4,0,0);
			//translate(x[i],y[i]);
		   ellipse(x[i],y[i],40,40);
			}
		//popMatrix();
		
		//line(v1.x, v1.y, v1.z, v2.x, v2.y, v2.z);
		popMatrix();
	}
	// -----------------------------------------------------------------

	// hand events

	public void onNewHand(SimpleOpenNI curContext, int handId, PVector pos)

	{

		println("onNewHand - handId: " + handId + ", pos: " + pos);

		ArrayList<PVector> vecList = new ArrayList<PVector>();

		vecList.add(pos);

		handPathList.put(handId, vecList);

	}

	public void onTrackedHand(SimpleOpenNI curContext, int handId, PVector pos)

	{

		// println("onTrackedHand - handId: " + handId + ", pos: " + pos );

		ArrayList<PVector> vecList = handPathList.get(handId);

		if (vecList != null)

		{

			vecList.add(0, pos);

			if (vecList.size() >= handVecListSize)

				// remove the last point

				vecList.remove(vecList.size() - 1);

		}

	}

	public void onLostHand(SimpleOpenNI curContext, int handId)

	{

		println("onLostHand - handId: " + handId);

		handPathList.remove(handId);

	}

	// -----------------------------------------------------------------

	// gesture events

	public void onCompletedGesture(SimpleOpenNI curContext, int gestureType,
			PVector pos)

	{

		println("onCompletedGesture - gestureType: " + gestureType + ", pos: "
				+ pos);

		int handId = context.startTrackingHand(pos);

		println("hand stracked: " + handId);

	}

	// -----------------------------------------------------------------

	// Keyboard event

	public void keyPressed()

	{

		switch (key)

		{

		case ' ':

			context.setMirror(!context.mirror());

			break;

		case '1':

			context.setMirror(true);

			break;

		case '2':

			context.setMirror(false);

			break;

		}

	}
	public void drawHUD() {
		image(context.depthImage(), 0, 0, 160, 120);

		loadPixels();
		// int[] userList = context.getUsers();

		updatePixels();

	}
}
